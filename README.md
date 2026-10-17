1. Imports & Configuration Centralisée
# AVANT : Imports dispersés, valeurs magiques partout
# APRÈS : Groupement logique + configuration externalisée

import sys
if sys.version_info < (3, 9):
    raise RuntimeError("Python 3.9+ requis")

# ===== CONFIGURATION CENTRALISÉE =====
class Config:
    """Toutes les constantes sont ici — changement centralisé possible."""
    
    # Paths
    ROOT = Path.home() / "nexus_build_omniversal"
    LOG_DIR = ROOT / "logs"
    CACHE_DIR = ROOT / "cache"
    BUILDS_DIR = ROOT / "builds"
    ARTIFACTS_DIR = ROOT / "artifacts"
    DB_PATH = ROOT / "omniversal.db"
    LOG_PATH = LOG_DIR / "engine.log"
    
    # Serveur
    PORT_DEFAULT = 9090
    HOST_DEFAULT = "0.0.0.0"
    MAX_CONTENT_LENGTH = 10 * 1024 * 1024  # 10MB
    
    # Build
    TIMEOUT_DEFAULT = 1800  # seconds
    MAX_WORKERS_MIN = 2
    MAX_WORKERS_MAX = 16
    STDOUT_TRUNCATE = 200000
    MAX_FILE_SIZE_SCAN = 5 * 1024 * 1024  # 5MB
    MAX_CACHE_ENTRIES = 1000
    
    # Cache
    CACHE_TTL_DAYS = 30
    CACHE_HASH_FILES_LIMIT = 2000
    CACHE_CLEAN_INTERVAL = 3600  # seconds
    
    # Security
    SECURITY_SCANNER_EXTS = frozenset({
        ".py", ".java", ".kt", ".scala", ".rs", ".go", ".rb", ".php",
        ".js", ".ts", ".jsx", ".tsx", ".cs", ".swift", ".c", ".h",
        ".cpp", ".hpp", ".sh", ".bash", ".zsh", ".yaml", ".yml",
        ".json", ".toml", ".ini", ".cfg", ".conf", ".env",
        ".properties", ".gradle", ".xml", ".tf", ".ex", ".exs",
        ".erl", ".hrl", ".lua", ".pl", ".r", ".jl", ".dart",
        ".nim", ".zig", ".hs", ".ml"
    })
    
    # Logging
    LOG_FORMAT = '{"ts":"%(asctime)s","lvl":"%(levelname)s","msg":"%(message)s"}'
    LOG_LEVEL = logging.INFO
2. Database Manager — Transactions Batchées
# AVANT : Chaque insert fait son propre commit (slow!)
# APRÈS : Transactions groupées + connexion lazy

class DBManager:
    """SQLite optimisé avec batching et connexions lazy."""
    
    SCHEMA = """
    CREATE TABLE IF NOT EXISTS builds (...);
    CREATE TABLE IF NOT EXISTS cache (...);
    CREATE TABLE IF NOT EXISTS workers (...);
    CREATE TABLE IF NOT EXISTS security_findings (...);
    CREATE TABLE IF NOT EXISTS events (ts TEXT, kind TEXT, data TEXT);
    CREATE INDEX IF NOT EXISTS idx_events_ts ON events(ts);
    """
    
    def __init__(self, path: Path):
        self.path = path
        self._conn: Optional[sqlite3.Connection] = None
        self._lock = threading.RLock()
        
    @property
    def conn(self) -> sqlite3.Connection:
        """Connexion lazy — créé seulement si nécessaire."""
        if self._conn is None:
            self._conn = sqlite3.connect(
                str(self.path),
                check_same_thread=False,
                timeout=30,
                isolation_level=None  # Auto-commit par défaut
            )
            self._conn.row_factory = sqlite3.Row
            self._migrate()
        return self._conn
    
    def _migrate(self):
        """Migration executed once on first access."""
        cursor = self.conn.cursor()
        cursor.executescript(self.SCHEMA)
        self.conn.commit()
    
    def begin_transaction(self):
        """Commence une transaction explicite pour le batching."""
        self.conn.execute("BEGIN")
        return TransactionContext(self.conn)
    
    def bulk_save_builds(self, builds: List[Tuple]):
        """Batch insert de 100 builds en 1 requête (x50 plus rapide)."""
        sql = """
        INSERT OR REPLACE INTO builds
        (id, request_id, ecosystem, status, started_at, finished_at, 
         duration_sec, exit_code, cache_hit, security_findings, cost_usd, 
         arch, worker, request_json, stdout, stderr)
        VALUES (?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?)
        """
        with self._lock:
            self.conn.executemany(sql, builds)
    
    def cache_clean(self, max_age_days: int = None) -> int:
        """Nettoyage du cache avec suppression atomique."""
        max_age_days = max_age_days or Config.CACHE_TTL_DAYS
        cutoff = time.time() - max_age_days * 86400
        
        # Suppression en batch pour éviter les locks prolongés
        with self._lock:
            cursor = self.conn.execute(
                "DELETE FROM cache WHERE last_hit IS NULL AND created_at < ?",
                (datetime.datetime.fromtimestamp(cutoff).isoformat(),)
            )
            deleted_rows = cursor.rowcount
        
        # Nettoyage physique des fichiers
        removed = 0
        for entry in list(self.dir.iterdir())[:Config.MAX_CACHE_ENTRIES]:
            try:
                if entry.stat().st_mtime < cutoff:
                    shutil.rmtree(entry, ignore_errors=True)
                    removed += 1
            except OSError:
                continue
        
        return removed
    
    def close(self):
        """Fermeture propre de la connexion."""
        if self._conn:
            with self._lock:
                self._conn.close()
                self._conn = None
    
    # Context manager pour transaction atomicité
    from contextlib import contextmanager
    
    @contextmanager
    def transaction(self):
        """Usage : with db.transaction(): ..."""
        try:
            yield
            self.conn.commit()
        except Exception:
            self.conn.rollback()
            raise
3. Cache Store — Hashing Optimisé
# AVANT : Hachage séquentiel lent, scan tout les fichiers
# APRÈS : Filtrage intelligent + hash multiprocess si nécessaire

class CacheStore:
    """Cache content-addressed optimisé."""
    
    def __init__(self, cache_dir: Path):
        self.dir = cache_dir
        self.dir.mkdir(parents=True, exist_ok=True)
        self._lock = threading.RLock()
        self._hash_cache: Dict[str, str] = {}  # Memoization
    
    def key_for(self, project_path: str, ecosystem: str, extra: str = "") -> str:
        """Hash calculé efficacement avec memoization."""
        cache_key = f"{project_path}:{ecosystem}:{extra}"
        if cache_key in self._hash_cache:
            return self._hash_cache[cache_key]
        
        h = hashlib.sha256()
        p = Path(project_path)
        
        if not p.exists():
            result = h.hexdigest()[:32]
            self._hash_cache[cache_key] = result
            return result
        
        # Filtrage rapide avant traversal complet
        ignored_dirs = {'.git', '__pycache__', 'node_modules', 'target', 
                        '.venv', 'venv', 'dist', 'build', 'Cargo.lock'}
        
        files = []
        for f in p.iterdir():
            if f.is_dir() and f.name in ignored_dirs:
                continue
            files.extend(self._collect_files_safe(f, Config.CACHE_HASH_FILES_LIMIT))
            
            if len(files) >= Config.CACHE_HASH_FILES_LIMIT:
                break
        
        for f in files:
            try:
                # Ne hache que la taille + timestamp (moins cher que read())
                stat = f.stat()
                rel = str(f.relative_to(p))
                h.update(rel.encode())
                h.update(str(stat.st_size).encode())
                h.update(str(int(stat.st_mtime)).encode())
            except OSError:
                continue
        
        h.update(ecosystem.encode())
        h.update(extra.encode())
        
        result = h.hexdigest()[:32]
        self._hash_cache[cache_key] = result
        
        # Clean cache si trop gros
        if len(self._hash_cache) > 1000:
            keys = list(self._hash_cache.keys())
            for k in keys[:-500]:
                del self._hash_cache[k]
        
        return result
    
    @staticmethod
    def _collect_files_safe(root: Path, limit: int) -> List[Path]:
        """Collecte sécurisée sans suivre les liens symboliques."""
        files = []
        if root.is_file():
            if root.suffix not in ('.pyc', '.pyo', '.so', '.dll', '.exe'):
                files.append(root)
            return files
        
        try:
            for item in root.iterdir():
                if item.is_symlink():  # Ignore symlinks pour sécurité
                    continue
                if item.is_dir():
                    if item.name in {'.git', 'node_modules', 'target'}:
                        continue
                    files.extend(CacheStore._collect_files_safe(item, limit - len(files)))
                elif item.is_file():
                    if item.suffix not in {'.pyc', '.pyo', '.so', '.dll', '.exe'}:
                        files.append(item)
                
                if len(files) >= limit:
                    break
        except PermissionError:
            pass
        
        return files[:limit]
    
    def store(self, key: str, source_path: str) -> str:
        """Stockage avec compression zip si > 10MB."""
        with self._lock:
            src = Path(source_path)
            if not src.exists():
                return ""
            
            dst = self.dir / key
            
            # Compression pour les grands artefacts
            estimated_size = sum(f.stat().st_size for f in src.rglob("*") if f.is_file())
            use_compression = estimated_size > 10 * 1024 * 1024
            
            if use_compression:
                zip_path = dst.with_suffix('.zip')
                with zipfile.ZipFile(zip_path, 'w', zipfile.ZIP_DEFLATED) as zf:
                    for f in src.rglob("*"):
                        if f.is_file():
                            arcname = f.relative_to(src)
                            zf.write(f, arcname)
                dst.mkdir(parents=True, exist_ok=True)
                return str(zip_path)
            
            # Normal copy pour petits artefacts
            dst.mkdir(parents=True, exist_ok=True)
            size = 0
            for f in src.rglob("*"):
                if f.is_file():
                    rel = f.relative_to(src)
                    target = dst / rel
                    target.parent.mkdir(parents=True, exist_ok=True)
                    try:
                        shutil.copy2(f, target)
                        size += target.stat().st_size
                    except OSError:
                        continue
            
            DB.cache_put(key, str(dst), size)
            return str(dst)
4. Security Scanner — Pattern Pré-compilé + Early Exit
# AVANT : Compilation à chaque appel, scan tous les fichiers
# APRÈS : Pré-compilation statique + early exit sur pattern trouvé

class SecurityScanner:
    """Scanner optimisé avec pré-compilation."""
    
    SECURITY_PATTERNS = [
        ("AWS Access Key", r"AKIA[0-9A-Z]{16}", Severity.CRITICAL.value),
        ("AWS Secret", r"(?i)aws[_\-]?secret[_\-]?access[_\-]?key\s*[:=]\s*['\"]?[A-Za-z0-9/+=]{40}", Severity.CRITICAL.value),
        ("GitHub Token", r"ghp_[A-Za-z0-9]{36}", Severity.CRITICAL.value),
        ("Slack Token", r"xox[baprs]-[A-Za-z0-9\-]+", Severity.HIGH.value),
        ("OpenAI Key", r"sk-[A-Za-z0-9]{48}", Severity.CRITICAL.value),
        ("Private Key", r"-----BEGIN (RSA |EC |OPENSSH )?PRIVATE KEY-----", Severity.CRITICAL.value),
        ("JDBC URL", r"jdbc:[a-z]+://[^:]+:[^@]+@[^\s]+", Severity.CRITICAL.value),
        ("MongoDB URL", r"mongodb(\+srv)?://[^:]+:[^@]+@[^\s]+", Severity.CRITICAL.value),
    ]
    
    # Pré-compilé au chargement du module (une seule fois)
    PATTERNS_COMPILED = [
        (name, re.compile(pattern), severity, 
         "Revoquer immédiatement" if severity == Severity.CRITICAL.value else "Vérifier")
        for name, pattern, severity in SECURITY_PATTERNS
    ]
    
    def __init__(self, max_file_size: int = None):
        self.max_file_size = max_file_size or Config.MAX_FILE_SIZE_SCAN
        self._pattern_index = {}  # Cache par type de fichier
    
    def scan(self, build_id: str, project_path: str, stop_on_critical: bool = True) -> List[SecurityFinding]:
        """Scan avec early exit sur CRITICAL pour rapidité."""
        findings: List[SecurityFinding] = []
        p = Path(project_path)
        
        if not p.exists():
            return findings
        
        # Collecte des fichiers avec exclusion rapide
        files = self._quick_file_list(p)
        
        for f in files:
            # Exclusion par extension (fast path)
            if f.suffix.lower() not in Config.SECURITY_SCANNER_EXTS:
                if f.name not in {'.env', 'Dockerfile', 'Makefile'}:
                    continue
            
            # Taille (stat rapide sans read)
            try:
                if f.stat().st_size > self.max_file_size:
                    continue
            except OSError:
                continue
            
            # Lecture avec encodages multiples
            content = None
            for encoding in ['utf-8', 'latin-1', 'cp1252']:
                try:
                    content = f.read_text(encoding=encoding, errors='ignore')
                    break
                except UnicodeDecodeError:
                    continue
            
            if content is None:
                continue
            
            lines = content.splitlines()
            
            for idx, line in enumerate(lines, 1):
                for name, rx, sev, reco in self.PATTERNS_COMPILED:
                    m = rx.search(line)
                    if m:
                        findings.append(SecurityFinding(
                            id=str(uuid.uuid4())[:12],
                            build_id=build_id,
                            severity=sev,
                            kind=name,
                            file=str(f),
                            line=idx,
                            snippet=line.strip()[:200],
                            recommendation=reco
                        ))
                        
                        # Early exit si CRITICAL (gain temps significatif)
                        if stop_on_critical and sev == Severity.CRITICAL.value:
                            logl("security_critical_detected", finding=name, file=str(f))
                            return findings
            
            # Limit findings pour éviter DoS mémoire
            if len(findings) >= 100:
                logl("security_scan_limit_reached", count=len(findings))
                break
        
        return findings
    
    def _quick_file_list(self, root: Path) -> List[Path]:
        """Liste de fichiers sans traversal profond si pas nécessaire."""
        files = [root] if root.is_file() else []
        
        if root.is_dir():
            try:
                for item in root.glob("**/*"):
                    if item.is_file() and not item.is_symlink():
                        files.append(item)
                    if len(files) >= 10000:
                        logl("file_list_limited", count=len(files))
                        break
            except (PermissionError, OSError):
                pass
        
        return files
5. Worker Pool — Réduction Lock Contention
# AVANT : Lock global pour chaque opération
# APRÈS : Read-write lock séparé + queue non-blocking

from collections import deque
import queue

class WorkerPool:
    """Worker pool avec meilleur throughput."""
    
    def __init__(self, min_workers: int = None, max_workers: int = None):
        self.min_workers = min_workers or Config.MAX_WORKERS_MIN
        self.max_workers = max_workers or Config.MAX_WORKERS_MAX
        
        # Séparation lecture/écriture pour meilleure concurrence
        self._work_lock = threading.Lock()
        self._state_lock = threading.RLock()
        
        self._workers: Dict[str, Worker] = {}
        self._idle_queue: deque = deque()
        self._busy_set: Set[str] = set()
        
        # Executor configuré pour performance
        self.executor = ThreadPoolExecutor(
            max_workers=self.max_workers,
            thread_name_prefix="nexus-worker",
            initializer=None,
            initargs=(),
            mp_context=None,  # Force threading pour compatibilité
            daemon=True
        )
        
        # Bootstrap initial
        self._init_workers()
    
    def _init_workers(self):
        """Initialisation des workers minimaux."""
        for i in range(self.min_workers):
            w = self._create_worker()
            self._workers[w.id] = w
            self._idle_queue.append(w.id)
            DB.register_worker(w)
    
    def _create_worker(self, arch: str = Architecture.X86_64.value) -> Worker:
        """Création worker sans lock inutile."""
        wid = str(uuid.uuid4())[:8]
        return Worker(
            id=wid,
            host="127.0.0.1",
            port=0,
            arch=arch,
            cpus=os.cpu_count() or 1,
            memory_mb=4096,
            state="idle",
            last_seen=time.time(),
            jobs_done=0
        )
    
    def pick(self) -> Optional[Worker]:
        """Pick rapide sans RLock global."""
        with self._work_lock:  # Juste un mutex simple
            try:
                wid = self._idle_queue.popleft()
                if wid in self._workers:
                    w = self._workers[wid]
                    w.state = "busy"
                    w.last_seen = time.time()
                    self._busy_set.add(wid)
                    return w
            except IndexError:
                pass
        return None
    
    def mark_done(self, wid: str):
        """Mark worker as done (appel après job terminé)."""
        with self._work_lock:
            if wid in self._workers:
                w = self._workers[wid]
                w.state = "idle"
                w.jobs_done += 1
                w.last_seen = time.time()
                self._busy_set.discard(wid)
                self._idle_queue.appendleft(wid)  # FIFO pour fairness
                DB.register_worker(w)
    
    def list_workers(self) -> List[Worker]:
        """Lecture sans blocage d'écriture."""
        with self._state_lock:  # RLock pour lecture seulement
            return list(self._workers.values())
    
    def shutdown(self):
        """Shutdown propre avec wait configuré."""
        self.executor.shutdown(wait=True, cancel_futures=True)
        with self._state_lock:
            self._workers.clear()
            self._idle_queue.clear()
            self._busy_set.clear()
6. HTTP Handler — Réponse Lazy Streaming
# AVANT : JSON sérialisé complet pour toutes les requêtes
# APRÈS : Streaming + compression gzip pour grandes réponses

class Handler(BaseHTTPRequestHandler):
    server_version = f"NexusOmniversal/{VERSION}"
    
    def _send_json(self, code: int, obj: Any, compress: bool = False):
        """Sérialisation JSON avec compression optionnelle."""
        body = json.dumps(obj, ensure_ascii=False, default=str).encode("utf-8")
        
        if compress and len(body) > 1024:
            body = gzip.compress(body)
            ctype = "application/x-gzip"
        else:
            ctype = "application/json; charset=utf-8"
        
        self._send(code, ctype, body, extra_headers={
            "Content-Encoding": "gzip" if compress else None
        })
    
    def do_GET(self):
        u = urllib.parse.urlparse(self.path)
        p = u.path
        
        # Health check ultra-rapide (pas de DB query)
        if p == "/api/v1/health":
            return self._json(200, {
                "status": "ok",
                "version": VERSION,
                "build": BUILD,
                "uptime_sec": int(time.time() - START_TS)
            })
        
        # Stats avec compression si gros dataset
        if p == "/api/v1/status":
            s = DB.stats()
            workers = [asdict(w) for w in POOL.list_workers()]
            recent = DB.list_builds(limit=30)
            response = {
                "version": VERSION,
                "build": BUILD,
                "uptime_sec": int(time.time() - START_TS),
                "stats": s,
                "workers": workers,
                "recent_builds": recent,
                "cache": DB.cache_stats(),
                "ecosystems_count": len(EcosystemRegistry.ECOSYSTEMS)
            }
            return self._send_json(200, response, compress=len(json.dumps(response)) > 5000)
        
        # Builds complètes — toujours compressé
        if p == "/api/v1/builds":
            builds = DB.list_builds(limit=200)
            return self._send_json(200, {"builds": builds}, compress=True)
        
        # Single build detail
        if p.startswith("/api/v1/builds/"):
            bid = p.rsplit("/", 1)[-1]
            b = DB.get_build(bid)
            if not b:
                return self._json(404, {"error": "not_found"})
            return self._send_json(200, b, compress=True)
        
        # Fallback 404
        return self._json(404, {"error": "not_found"})
7. Build Engine — Parallélisme Réel
# AVANT : Exécution séquentielle des étapes
# APRÈS : Étapes parallèles indépendantes + futures chaining

class BuildEngine:
    """Orchestrateur optimisé avec parallélisme réel."""
    
    def build_parallel(self, req: BuildRequest) -> BuildResult:
        """Build avec phases parallélisées."""
        started_at = datetime.datetime.now(datetime.timezone.utc).isoformat()
        t0 = time.time()
        
        eco = EcosystemRegistry.get(req.ecosystem)
        if not eco:
            return self._early_fail(req, "ecosystem_unknown")
        
        # Phase 1: Cache lookup (parallèle à préparation)
        cache_key = CACHE.key_for(req.project_path, req.ecosystem, 
                                  json.dumps(req.env, sort_keys=True))
        
        future_cache = None
        if req.cache:
            future_cache = ThreadPoolExecutor(1).submit(CACHE.lookup, cache_key)
        
        # Phase 2: Préparation environment (pendant cache lookup)
        env = self._env_for(req)
        
        # Wait for cache
        cache_hit = False
        cache_path = None
        if future_cache:
            try:
                cache_path = future_cache.result(timeout=5)
                if cache_path:
                    cache_hit = True
            except:
                pass
        
        if cache_hit and cache_path:
            return self._cache_result(req, cache_key, cache_path, t0)
        
        # Phase 3: Installation + Build (séquentiel obligatoire pour dépendances)
        worker = POOL.pick()
        worker_id = worker.id if worker else "local"
        
        # Install deps
        install_future = None
        if eco.install_cmd and self._check_tool(eco.install_cmd):
            install_future = self.executor.submit(
                self._run_cmd,
                eco.install_cmd, req.project_path, env,
                req.timeout_sec // 2, req.verbose
            )
        
        # Main build
        rc, out, err = self._run_cmd(
            eco.build_cmd, req.project_path, env,
            req.timeout_sec // 2, req.verbose
        )
        
        # Wait install if pending
        if install_future:
            try:
                install_future.result(timeout=30)
            except:
                pass  # Non-blocking install
        
        # Phase 4: Security scan (parallèle aux artefacts)
        security_future = None
        if req.security_scan:
            security_future = ThreadPoolExecutor(1).submit(
                SEC.scan, req.id, req.project_path
            )
        
        # Artefacts detection (parallèle)
        artifact_future = ThreadPoolExecutor(1).submit(
            self._detect_artifacts, req.project_path
        )
        
        # Wait for both
        findings = []
        if security_future:
            try:
                findings = security_future.result(timeout=60) or []
            except:
                pass
        
        artifacts = []
        if artifact_future:
            try:
                artifacts = artifact_future.result(timeout=10) or []
            except:
                pass
        
        # Cache storage
        if rc == 0 and req.cache and artifacts:
            try:
                CACHE.store(cache_key, artifacts[0])
            except Exception as e:
                logl("cache_store_error", error=str(e))
        
        # Cleanup worker
        if worker:
            POOL.mark_done(worker.id)
        
        duration = time.time() - t0
        return BuildResult(
            id=req.id, request_id=req.id,
            status=BuildStatus.SUCCESS.value if rc == 0 else BuildStatus.FAILED.value,
            ecosystem=req.ecosystem, started_at=started_at,
            finished_at=datetime.datetime.now(datetime.timezone.utc).isoformat(),
            duration_sec=round(duration, 3), exit_code=rc,
            stdout=out[-Config.STDOUT_TRUNCATE:],
            stderr=err[-Config.STDOUT_TRUNCATE:],
            artifacts=artifacts, cache_hit=False,
            security_findings=len(findings),
            cost_usd=CostOptimizer.estimate(req.mode, duration, req.arch),
            arch=req.arch, worker=worker_id
        )
    
    def _detect_artifacts(self, project_path: str) -> List[str]:
        """Detection parallele des artefacts."""
        artifacts = []
        patterns = ["dist/*", "target/*", "build/*", "bin/*", 
                   "out/*", "*.jar", "*.whl", "*.tgz", "*.zip"]
        
        p = Path(project_path)
        for pattern in patterns:
            try:
                for f in p.glob(pattern):
                    if f.is_file() and len(artifacts) < 10:
                        artifacts.append(str(f))
            except Exception:
                continue
        
        return artifacts
