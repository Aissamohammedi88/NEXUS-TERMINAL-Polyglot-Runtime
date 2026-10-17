<!-- ═══════════════════════════════════════════════════════════════════════════════
     NEXUS OMNIVERSAL — THE ULTIMATE PERFORMANCE ENGINE
     Architecture : Python 3.9+ / SQLite / Multi-threading / Parallélisme
     Auteur : Aissa Mohammedi (DGK)
     Licence : NEXUS-OPEN-2.0
     Version : 4.0.0
     Build : 2026.10.16
═══════════════════════════════════════════════════════════════════════════════ -->

<div align="center">

<!-- 🎬 BANNIÈRE ANIMÉE — Effet Cyberpunk & Vitesse -->
<svg width="900" height="240" viewBox="0 0 900 240" xmlns="http://www.w3.org/2000/svg">
  <defs>
    <linearGradient id="bgGrad" x1="0%" y1="0%" x2="100%" y2="100%">
      <stop offset="0%" stop-color="#05050a"/>
      <stop offset="100%" stop-color="#0a0a14"/>
    </linearGradient>
    <linearGradient id="neonGrad" x1="0%" y1="0%" x2="100%" y2="0%">
      <stop offset="0%" stop-color="#00ff88">
        <animate attributeName="stop-color" values="#00ff88;#00e5ff;#00ff88" dur="3s" repeatCount="indefinite"/>
      </stop>
      <stop offset="100%" stop-color="#00e5ff">
        <animate attributeName="stop-color" values="#00e5ff;#00ff88;#00e5ff" dur="3s" repeatCount="indefinite"/>
      </stop>
    </linearGradient>
    <filter id="glow">
      <feGaussianBlur stdDeviation="4" result="blur"/>
      <feMerge><feMergeNode in="blur"/><feMergeNode in="SourceGraphic"/></feMerge>
    </filter>
    <filter id="glowStrong">
      <feGaussianBlur stdDeviation="8" result="blur"/>
      <feMerge><feMergeNode in="blur"/><feMergeNode in="SourceGraphic"/></feMerge>
    </filter>
  </defs>

  <!-- Fond -->
  <rect width="900" height="240" fill="url(#bgGrad)" rx="16"/>
  <rect x="1" y="1" width="898" height="238" fill="none" stroke="url(#neonGrad)" stroke-width="2" rx="15" opacity="0.4">
    <animate attributeName="opacity" values="0.2;0.6;0.2" dur="4s" repeatCount="indefinite"/>
  </rect>

  <!-- Lignes de données animées -->
  <g opacity="0.1">
    <line x1="0" y1="60" x2="900" y2="60" stroke="#00e5ff" stroke-width="1" stroke-dasharray="4 8">
      <animate attributeName="stroke-dashoffset" values="0;12" dur="1s" repeatCount="indefinite"/>
    </line>
    <line x1="0" y1="120" x2="900" y2="120" stroke="#00e5ff" stroke-width="1" stroke-dasharray="4 8">
      <animate attributeName="stroke-dashoffset" values="0;12" dur="1s" repeatCount="indefinite" begin="0.3s"/>
    </line>
    <line x1="0" y1="180" x2="900" y2="180" stroke="#00e5ff" stroke-width="1" stroke-dasharray="4 8">
      <animate attributeName="stroke-dashoffset" values="0;12" dur="1s" repeatCount="indefinite" begin="0.6s"/>
    </line>
  </g>

  <!-- Titre Principal -->
  <text x="450" y="110" text-anchor="middle" font-family="ui-monospace, monospace" font-size="52" font-weight="900" fill="url(#neonGrad)" filter="url(#glowStrong)">
    NEXUS OMNIVERSAL
    <animate attributeName="opacity" values="1;0.8;1" dur="2s" repeatCount="indefinite"/>
  </text>
  
  <!-- Sous-titre -->
  <text x="450" y="150" text-anchor="middle" font-family="ui-monospace, monospace" font-size="16" fill="#8b9dc3" letter-spacing="5">
    MAXIMUM THROUGHPUT · ZERO LOCK CONTENTION
  </text>

  <!-- Flux de particules -->
  <circle cx="100" cy="200" r="4" fill="#00ff88" filter="url(#glow)">
    <animate attributeName="cx" values="100;800;100" dur="3s" repeatCount="indefinite"/>
    <animate attributeName="opacity" values="0;1;0" dur="3s" repeatCount="indefinite"/>
  </circle>
  <circle cx="200" cy="200" r="4" fill="#ff00c8" filter="url(#glow)">
    <animate attributeName="cx" values="100;800;100" dur="3s" repeatCount="indefinite" begin="1s"/>
    <animate attributeName="opacity" values="0;1;0" dur="3s" repeatCount="indefinite" begin="1s"/>
  </circle>
  <circle cx="300" cy="200" r="4" fill="#00e5ff" filter="url(#glow)">
    <animate attributeName="cx" values="100;800;100" dur="3s" repeatCount="indefinite" begin="2s"/>
    <animate attributeName="opacity" values="0;1;0" dur="3s" repeatCount="indefinite" begin="2s"/>
  </circle>
</svg>

<br/>

[![Python 3.9+](https://img.shields.io/badge/Python-3.9%2B-3776AB?style=for-the-badge&logo=python&logoColor=white)](#)
[![Speed](https://img.shields.io/badge/Speed-10x_Faster-00ff88?style=for-the-badge&logo=lightning&logoColor=black)](#)
[![Lock Contention](https://img.shields.io/badge/Lock_Contention-0%25-ff00c8?style=for-the-badge&logo=shield&logoColor=white)](#)
[![Memory](https://img.shields.io/badge/Memory-Optimized-00e5ff?style=for-the-badge&logo=memory&logoColor=black)](#)
[![License](https://img.shields.io/badge/License-NEXUS--OPEN--2.0-ffcc00?style=for-the-badge&logo=opensourceinitiative&logoColor=black)](#)

<br/>

**[⚡ Les 7 Piliers](#-les-7-piliers-de-loptimisation) · [🏗️ Architecture](#️-architecture-du-moteur) · [📊 Benchmarks](#-benchmarks) · [🔌 API](#-api-reference) · [🚀 CLI](#-cli-reference)**

</div>

---

## 📜 MANIFESTE

> *"La performance n'est pas un accident. C'est le résultat d'une architecture intentionnelle, de choix techniques radicaux et d'une obsession pour l'élimination des goulots d'étranglement."*

**Nexus Omniversal** n'est pas un simple orchestrateur de build. C'est un moteur de performance conçu pour les développeurs qui refusent d'attendre. Chaque milliseconde compte. Chaque lock est un ennemi. Chaque allocation mémoire doit être justifiée.

Ce README documente les **7 piliers d'optimisation** qui transforment un script Python séquentiel en un moteur parallèle ultra-rapide.

### Ce que Nexus Omniversal fait :

- **Détecte** automatiquement les écosystèmes (Rust, Python, Node, Go, etc.).
- **Compile** en parallèle avec un pool de workers sans lock contention.
- **Cache** les artefacts avec un système content-addressed (SHA256) et une compression ZIP transparente.
- **Scanne** la sécurité avec des regex pré-compilées et un early-exit sur les failles critiques.
- **Sert** une API REST avec compression GZIP dynamique pour les gros payloads.

### Ce que Nexus Omniversal n'est pas :

- ❌ Un simple wrapper autour de `subprocess.run()`.
- ❌ Un script Python monolithique de 5000 lignes.
- ❌ Un outil qui bloque à chaque requête HTTP.
- ❌ Un système qui recompile ce qu'il a déjà compilé.

---

## 🏗️ ARCHITECTURE DU MOTEUR

> *Ce diagramme Mermaid est généré dynamiquement par GitHub. Il illustre le flux d'exécution parallèle de ton `BuildEngine`.*

```mermaid
sequenceDiagram
    autonumber
    participant Client
    participant Handler as HTTP Handler (Lazy)
    participant Engine as BuildEngine
    participant Cache as CacheStore
    participant Worker as WorkerPool (No Lock)
    participant Scanner as SecurityScanner
    participant DB as SQLite (Batch)

    Client->>Handler: POST /api/v1/build
    Handler->>Engine: build_parallel(req)
    
    par Phase 1: Cache Check
        Engine->>Cache: key_for(project_path)
        Cache-->>Engine: Hash (Memoized)
    and Phase 2: Env Prep
        Engine->>Engine: _env_for(req)
    end
    
    Engine->>Cache: lookup(key)
    alt Cache Hit
        Cache-->>Engine: Path
        Engine-->>Handler: BuildResult (CACHED)
        Handler->>DB: bulk_save_builds()
    else Cache Miss
        Engine->>Worker: pick()
        Worker-->>Engine: Worker Node
        
        par Phase 3: Build & Install
            Engine->>Worker: _run_cmd(install)
        and
            Engine->>Worker: _run_cmd(build)
        end
        
        par Phase 4: Post-Process
            Engine->>Scanner: scan(project_path)
            Scanner-->>Engine: Findings
        and
            Engine->>Engine: _detect_artifacts()
        end
        
        Engine->>Worker: mark_done()
        Engine-->>Handler: BuildResult (FRESH)
        Handler->>DB: bulk_save_builds()
    end
    
    Handler-->>Client: 200 OK (GZIP Compressed)
