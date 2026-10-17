# NEXUS-TERMINAL-Polyglot-Runtime
ncludes native support for interpreted languages (Python,   Ruby, Perl, PHP, JavaScript, Lua, Tcl, R, Julia), compiled languages   (C, C++, Rust, Go, Zig, Nim, D, Swift, Objective-C), JVM languages   (Java, Kotlin, Scala, Groovy, Clojure), .NET languages (C#, F#, VB.NET),   functional languages (Haskell, OCaml, Erlang, Elixir, F#, Scheme, Lisp), 
🌐 NEXUS TERMINAL — Universal Polyglot Runtime
<div align="center"><!-- ═══════════════════════════════════════════════════════════════════════════ HERO ANIMATION — SMIL SVG (Fonctionne nativement sur GitHub) ═══════════════════════════════════════════════════════════════════════════ --><svg width="100%" height="auto" viewBox="0 0 1000 400" xmlns="http://www.w3.org/2000/svg" style="max-width: 900px; border-radius: 16px; background: #05070d; box-shadow: 0 0 40px rgba(0, 217, 255, 0.2);"> <defs> <linearGradient id="nexusGrad" x1="0%" y1="0%" x2="100%" y2="100%"> <stop offset="0%" stop-color="#00d9ff"> <animate attributeName="stop-color" values="#00d9ff;#a78bfa;#ff2d95;#00d9ff" dur="10s" repeatCount="indefinite"/> </stop> <stop offset="100%" stop-color="#a78bfa"> <animate attributeName="stop-color" values="#a78bfa;#ff2d95;#00d9ff;#a78bfa" dur="10s" repeatCount="indefinite"/> </stop> </linearGradient> <filter id="glow"> <feGaussianBlur stdDeviation="4" result="coloredBlur"/> <feMerge> <feMergeNode in="coloredBlur"/> <feMergeNode in="SourceGraphic"/> </feMerge> </filter> <!-- Motif de grille --> <pattern id="grid" width="40" height="40" patternUnits="userSpaceOnUse"> <path d="M 40 0 L 0 0 0 40" fill="none" stroke="#1a2333" stroke-width="1"/> </pattern> </defs> <!-- Fond --> <rect width="1000" height="400" fill="url(#grid)" /> <rect width="1000" height="400" fill="rgba(5, 7, 13, 0.8)" /> <!-- Connexions entre les nœuds --> <g stroke="url(#nexusGrad)" stroke-width="2" stroke-dasharray="5,5" opacity="0.4"> <line x1="500" y1="200" x2="200" y2="100"> <animate attributeName="stroke-dashoffset" values="0;20" dur="1s" repeatCount="indefinite"/> </line> <line x1="500" y1="200" x2="800" y2="100"> <animate attributeName="stroke-dashoffset" values="0;20" dur="1.2s" repeatCount="indefinite"/> </line> <line x1="500" y1="200" x2="200" y2="300"> <animate attributeName="stroke-dashoffset" values="0;20" dur="0.8s" repeatCount="indefinite"/> </line> <line x1="500" y1="200" x2="800" y2="300"> <animate attributeName="stroke-dashoffset" values="0;20" dur="1.5s" repeatCount="indefinite"/> </line> </g> <!-- Nœuds périphériques (Langages) --> <g font-family="ui-monospace, monospace" font-size="14" fill="#e0e7ff" text-anchor="middle"> <!-- Python --> <circle cx="200" cy="100" r="30" fill="#0f1520" stroke="#00d9ff" stroke-width="2" filter="url(#glow)"> <animate attributeName="r" values="30;35;30" dur="2s" repeatCount="indefinite"/> </circle> <text x="200" y="105" fill="#00d9ff">PY</text> <!-- Rust --> <circle cx="800" cy="100" r="30" fill="#0f1520" stroke="#ff2d95" stroke-width="2" filter="url(#glow)"> <animate attributeName="r" values="30;35;30" dur="2.3s" repeatCount="indefinite"/> </circle> <text x="800" y="105" fill="#ff2d95">RS</text> <!-- Go --> <circle cx="200" cy="300" r="30" fill="#0f1520" stroke="#a78bfa" stroke-width="2" filter="url(#glow)"> <animate attributeName="r" values="30;35;30" dur="1.8s" repeatCount="indefinite"/> </circle> <text x="200" y="305" fill="#a78bfa">GO</text> <!-- C/C++ --> <circle cx="800" cy="300" r="30" fill="#0f1520" stroke="#00ff88" stroke-width="2" filter="url(#glow)"> <animate attributeName="r" values="30;35;30" dur="2.5s" repeatCount="indefinite"/> </circle> <text x="800" y="305" fill="#00ff88">C++</text> </g> <!-- Nœud Central (Nexus) --> <circle cx="500" cy="200" r="70" fill="#0a0f1a" stroke="url(#nexusGrad)" stroke-width="4" filter="url(#glow)"> <animate attributeName="r" values="70;75;70" dur="3s" repeatCount="indefinite"/> </circle> <!-- Texte Central --> <text x="500" y="190" text-anchor="middle" font-family="ui-monospace, monospace" font-size="24" font-weight="bold" fill="#ffffff" filter="url(#glow)"> NEXUS </text> <text x="500" y="215" text-anchor="middle" font-family="ui-monospace, monospace" font-size="12" fill="#4a5878" letter-spacing="2"> RUNTIME </text> <!-- Ligne Scan --> <line x1="0" y1="380" x2="1000" y2="380" stroke="url(#nexusGrad)" stroke-width="2"> <animate attributeName="x1" values="0;1000;0" dur="8s" repeatCount="indefinite"/> </line> </svg>
The Smallest Sovereign Language Host

https://img.shields.io/badge/VERSION-1.0.0-00d9ff?style=for-the-badge&logo=terminal&logoColor=white
https://img.shields.io/badge/LANGUAGES-40%2B-a78bfa?style=for-the-badge&logo=code&logoColor=white
https://img.shields.io/badge/RUNTIMES-150%2B-ff2d95?style=for-the-badge&logo=cpu&logoColor=white
https://img.shields.io/badge/LICENSE-NEXUS--OPEN--2.0-00ff88?style=for-the-badge

https://img.shields.io/badge/STATUS-OPERATIONAL-success?style=flat-square
https://img.shields.io/badge/PLATFORM-Linux%20%7C%20macOS%20%7C%20WSL2-blue?style=flat-square
https://img.shields.io/badge/SOVEREIGN-100%25-red?style=flat-square

📖 Documentation · 🌐 Langages · ⚡ Architecture · 🚀 Installation
</div>
🎬 Simulation d'exécution (Live Terminal)
<div align="center"> <!-- Simulation de terminal via SVG pur --> <svg width="100%" height="auto" viewBox="0 0 800 300" xmlns="http://www.w3.org/2000/svg" style="max-width: 800px; border-radius: 12px; box-shadow: 0 10px 30px rgba(0,0,0,0.5); font-family: ui-monospace, monospace;"> <rect width="800" height="300" fill="#0a0f1a" rx="12"/> <rect width="800" height="40" fill="#131825" rx="12"/> <rect width="800" height="40" fill="#131825" /> <!-- Boutons mac style --> <circle cx="20" cy="20" r="6" fill="#ff5f57"/> <circle cx="40" cy="20" r="6" fill="#ffbd2e"/> <circle cx="60" cy="20" r="6" fill="#28ca42"/>

<text x="400" y="25" text-anchor="middle" fill="#4a5878" font-size="12">nexus-terminal — ~/projects</text>
<!-- Ligne 1 -->

<text x="20" y="70" fill="#00d9ff" font-size="14">➜ ~ </text>
<text x="60" y="70" fill="#e0e7ff" font-size="14">nexus run script.py</text>
<!-- Ligne 2 -->

<text x="20" y="100" fill="#a78bfa" font-size="14">[detect] </text>
<text x="90" y="100" fill="#4a5878" font-size="14">Language: Python 3.12 (shebang signature)</text>
<!-- Ligne 3 -->

<text x="20" y="130" fill="#a78bfa" font-size="14">[resolve] </text>
<text x="90" y="130" fill="#4a5878" font-size="14">Runtime: /usr/bin/python3.12 (cached)</text>
<!-- Ligne 4 -->

<text x="20" y="160" fill="#ff2d95" font-size="14">[sandbox] </text>
<text x="100" y="160" fill="#4a5878" font-size="14">Network blocked, FS restricted to /home/user</text>
<!-- Ligne 5 -->

<text x="20" y="190" fill="#00ff88" font-size="14">[execute] </text>
<text x="100" y="190" fill="#00ff88" font-size="14">Hello, world. (4.2ms)</text>
<!-- Ligne 6 -->

<text x="20" y="230" fill="#00d9ff" font-size="14">➜ ~ </text>
<text x="60" y="230" fill="#e0e7ff" font-size="14">nexus run main.rs</text>
<!-- Ligne 7 -->

<text x="20" y="260" fill="#a78bfa" font-size="14">[detect] </text>
<text x="90" y="260" fill="#4a5878" font-size="14">Language: Rust 1.75 (extension)</text>
<!-- Curseur clignotant --> <rect x="20" y="280" width="10" height="15" fill="#00d9ff"> <animate attributeName="opacity" values="1;0;1" dur="1s" repeatCount="indefinite"/> </rect> </svg> </div>
🏆 Pourquoi Nexus Terminal ?
<div align="center">
😤 Les autres	👑 Nexus Terminal
Tu tapes python3, gcc, cargo, java...	Tu tapes nexus run
Tu configures chaque projet différemment	Détection automatique par shebang
Ton code tourne sans isolation	Sandbox par défaut (réseau bloqué)
Dépendance au cloud	100% Souverain (zéro télémétrie)
5 étapes pour installer un runtime	Un seul binaire orchestre tout
</div>
🌐 40+ Langages Supportés
<div align="center"><details> <summary><b>🔥 Cliquez pour voir les 150+ runtimes supportés</b></summary> <br>
Catégorie	Langages	Runtimes
🐍 Interprétés	Python, Ruby, Perl, PHP, Lua, Tcl, R, Julia, Node.js, Deno, Bun	python3, ruby, perl, php, lua, tclsh, Rscript, julia, node, deno, bun
⚙️ Compilés	C, C++, Rust, Go, Zig, Nim, D, Swift, Objective-C, Ada, Fortran, Pascal, Crystal, V, Odin, Hare	gcc, clang, tcc, g++, rustc, cargo, go, zig, nim, dmd, swiftc, gnat, gfortran, fpc, crystal, v, odin, hare
☕ JVM	Java, Kotlin, Scala, Groovy, Clojure, JRuby, Jython	javac, kotlinc, scalac, groovy, clojure, jruby, jython
🔷 .NET	C#, F#, VB.NET, PowerShell	dotnet, mono, fsharpc, vbc, pwsh
🧠 Fonctionnels	Haskell, OCaml, Erlang, Elixir, F#, Elm, PureScript	ghc, cabal, stack, ocaml, dune, erlc, rebar3, elixirc, mix, elm, purs
📜 Scripting	Bash, Zsh, Fish, Ksh, Dash, AWK, sed, VimScript, Neovim, Nushell, Oil, Elvish, Xonsh	bash, zsh, fish, ksh, dash, awk, sed, vim, nvim, nu, oil, elvish, xonsh
🔧 Système / ASM	x86, x86_64, ARM, ARM64, RISC-V, WASM	nasm, gas, riscv-gcc, wat2wasm
📊 Données / Config	YAML, JSON, TOML, SQL, GraphQL, Protobuf	jq, yq, sqlite3, psql, mysql, graphql, protoc
📄 Markup	Markdown, LaTeX, AsciiDoc, reStructuredText	pandoc, glow, pdflatex, xelatex, asciidoctor
☁️ Domain Specific	Terraform, Ansible, Dockerfile, Makefile, CMake, Bazel, Nix	terraform, ansible, docker, make, cmake, bazel, nix
</details></div>
