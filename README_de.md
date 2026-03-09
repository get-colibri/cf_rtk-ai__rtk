<p align="center">
  <img src="https://avatars.githubusercontent.com/u/258253854?v=4" alt="RTK - Rust Token Killer" width="500">
</p>

<p align="center">
  <strong>High-performance CLI proxy that reduces LLM token consumption by 60-90%</strong>
</p>

<p align="center">
  <a href="https://github.com/rtk-ai/rtk/actions"><img src="https://github.com/rtk-ai/rtk/workflows/Security%20Check/badge.svg" alt="CI"></a>
  <a href="https://github.com/rtk-ai/rtk/releases"><img src="https://img.shields.io/github/v/release/rtk-ai/rtk" alt="Release"></a>
  <a href="https://opensource.org/licenses/MIT"><img src="https://img.shields.io/badge/License-MIT-yellow.svg" alt="License: MIT"></a>
  <a href="https://discord.gg/gFwRPEKq4p"><img src="https://img.shields.io/discord/1478373640461488159?label=Discord&logo=discord" alt="Discord"></a>
  <a href="https://formulae.brew.sh/formula/rtk"><img src="https://img.shields.io/homebrew/v/rtk" alt="Homebrew"></a>
</p>

<p align="center">
  <a href="https://www.rtk-ai.app">Website</a> &bull;
  <a href="#installation">Install</a> &bull;
  <a href="docs/TROUBLESHOOTING.md">Troubleshooting</a> &bull;
  <a href="ARCHITECTURE.md">Architecture</a> &bull;
  <a href="https://discord.gg/gFwRPEKq4p">Discord</a>
</p>

<p align="center">
  <a href="README.md">English</a> &bull;
  <a href="README_fr.md">Francais</a> &bull;
  <a href="README_zh.md">中文</a> &bull;
  <a href="README_ja.md">日本語</a> &bull;
  <a href="README_ko.md">한국어</a> &bull;
  <a href="README_es.md">Espanol</a> &bull;
  <a href="README_de.md">Deutsch</a>
</p>

---

rtk filtert und komprimiert Befehlsausgaben, bevor sie Ihren LLM-Kontext erreichen. Einzelnes Rust-Binary, null Abhängigkeiten, <10ms Overhead.
## Token-Einsparungen (30-minütige Claude Code-Sitzung)

| Operation                 | Häufigkeit | Standard     | rtk         | Ersparnis |
| ------------------------- | ---------- | ------------ | ----------- | --------- |
| `ls` / `tree`             | 10x        | 2.000        | 400         | -80%      |
| `cat` / `read`            | 20x        | 40.000       | 12.000      | -70%      |
| `grep` / `rg`             | 8x         | 16.000       | 3.200       | -80%      |
| `git status`              | 10x        | 3.000        | 600         | -80%      |
| `git diff`                | 5x         | 10.000       | 2.500       | -75%      |
| `git log`                 | 5x         | 2.500        | 500         | -80%      |
| `git add/commit/push`     | 8x         | 1.600        | 120         | -92%      |
| `cargo test` / `npm test` | 5x         | 25.000       | 2.500       | -90%      |
| `ruff check`              | 3x         | 3.000        | 600         | -80%      |
| `pytest`                  | 4x         | 8.000        | 800         | -90%      |
| `go test`                 | 3x         | 6.000        | 600         | -90%      |
| `docker ps`               | 3x         | 900          | 180         | -80%      |
| **Gesamt**                |            | **~118.000** | **~23.900** | **-80%**  |

> Schätzungen basierend auf mittelgroßen TypeScript/Rust-Projekten. Tatsächliche Ersparnisse variieren je nach Projektgröße.
## Installation

### Homebrew (empfohlen)


```bash
brew install rtk
```

### Schnellinstallation (Linux/macOS)


```bash
curl -fsSL https://raw.githubusercontent.com/rtk-ai/rtk/refs/heads/master/install.sh | sh
```

> Installiert nach `~/.local/bin`. Bei Bedarf zum PATH hinzufügen:

echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc  # or ~/.zshrc

### <br><br>Cargo


```bash
cargo install --git https://github.com/rtk-ai/rtk
```

### Vorkompilierte Binaries

Herunterladen von [Releases](https://github.com/rtk-ai/rtk/releases):

- macOS: `rtk-x86_64-apple-darwin.tar.gz` / `rtk-aarch64-apple-darwin.tar.gz`
- Linux: `rtk-x86_64-unknown-linux-musl.tar.gz` / `rtk-aarch64-unknown-linux-gnu.tar.gz`
- Windows: `rtk-x86_64-pc-windows-msvc.zip`

### Installation überprüfen


```bash
rtk --version # Sollte "rtk 0.27.1" zeigen
rtk gain # Zeigt Token-Einsparungsstatistiken
```

> **Warnung vor Namenskonflikt**: Es existiert ein anderes Projekt namens "rtk" (Rust Type Kit) auf crates.io. Wenn `rtk gain` fehlschlägt, haben Sie das falsche Paket. Verwenden Sie stattdessen `cargo install --git`.
## Schnellstart


```bash
# 1. Hook für Claude Code installieren (empfohlen)
rtk init --global
# Anweisungen folgen, um in ~/.claude/settings.json zu registrieren
# 2. Claude Code neu starten, dann testen
git status # Automatisch umgeschrieben zu rtk git status
```

Der Hook überschreibt Befehle transparent (z.B. `git status` -> `rtk git status`) vor der Ausführung. Claude sieht die Umschreibung nicht, es erhält nur die komprimierte Ausgabe.

## Funktionsweise


```
Ohne rtk:                  Mit rtk:
Claude --git status--> shell --> git    Claude --git status--> RTK --> git
^                                   |    ^                              |
| ~2.000 Token (roh)                |    | ~200 Token                   | filter
+-----------------------------------+    +-------(gefiltert)------------+----------+
```

Vier Strategien pro Befehlstyp:

1. **Smart Filtering** - Entfernt Rauschen (Kommentare, Whitespace, Boilerplate)
2. **Grouping** - Aggregiert ähnliche Elemente (Dateien nach Verzeichnis, Fehler nach Typ)
3. **Truncation** - Behält relevanten Kontext, kürzt Redundanz
4. **Deduplication** - Fasst wiederholte Log-Zeilen mit Zählern zusammen

## Befehle

### Dateien


```bash
rtk ls . # Token-optimierter Verzeichnisbaum
rtk read file.rs # Intelligente Dateilesung
rtk read file.rs -l aggressive # Nur Signaturen (entfernt Körper)
rtk smart file.rs # 2-Zeilen-Heuristik-Code-Zusammenfassung
rtk find "*.rs" . # Kompakte Suchergebnisse
rtk grep "pattern" . # Gruppierte Suchergebnisse
rtk diff file1 file2 # Komprimierte Diff
```

### Git


```bash
rtk git status # Kompakter Status
rtk git log -n 10 # Einzeilige Commits
rtk git diff # Komprimierte Diff
rtk git add # -> "ok"
rtk git commit -m "msg" # -> "ok abc1234"
rtk git push # -> "ok main"
rtk git pull # -> "ok 3 Dateien +10 -2"
```

### GitHub CLI


```bash
rtk gh pr list # Kompakte PR-Liste
rtk gh pr view 42 # PR-Details + Checks
rtk gh issue list # Kompakte Issue-Liste
rtk gh run list # Workflow-Run-Status
```

### Test-Runner


```bash
rtk test cargo test # Zeigt nur Fehler (-90%)
rtk err npm run build # Nur Fehler/Warnungen
rtk vitest run # Vitest kompakt (nur Fehler)
rtk playwright test # E2E-Ergebnisse (nur Fehler)
rtk pytest # Python-Tests (-90%)
rtk go test # Go-Tests (NDJSON, -90%)
rtk cargo test # Cargo-Tests (-90%)
```

### Build & Lint


```bash
rtk lint # ESLint gruppiert nach Regel/Datei
rtk lint biome # Unterstützt andere Linter
rtk tsc # TypeScript-Fehler gruppiert nach Datei
rtk next build # Next.js-Build kompakt
rtk prettier --check . # Dateien, die Formatierung brauchen
rtk cargo build # Cargo-Build (-80%)
rtk cargo clippy # Cargo clippy (-80%)
rtk ruff check # Python-Linting (JSON, -80%)
rtk golangci-lint run # Go-Linting (JSON, -85%)
```

### Paketmanager


```bash
rtk pnpm list # Kompakte Abhängigkeitsstruktur
rtk pip list # Python-Pakete (erkennt automatisch uv)
rtk pip outdated # Veraltete Pakete
rtk prisma generate # Schema-Generierung (keine ASCII-Kunst)
```

### Container


```bash
rtk docker ps # Kompakte Container-Liste
rtk docker images # Kompakte Image-Liste
rtk docker logs # Deduplizierte Logs
rtk docker compose ps # Compose-Services
rtk kubectl pods # Kompakte Pod-Liste
rtk kubectl logs # Deduplizierte Logs
rtk kubectl services # Kompakte Service-Liste
```

### Daten & Analytics


```bash
rtk json config.json # Struktur ohne Werte
rtk deps # Abhängigkeitsübersicht
rtk env -f AWS # Gefilterte Umgebungsvariablen
rtk log app.log # Deduplizierte Logs
rtk curl # Automatische JSON + Schema-Erkennung
rtk wget # Download, entfernt Fortschrittsbalken
rtk summary # Heuristische Zusammenfassung
rtk proxy # Raw-Passthrough + Tracking
```

### Token-Einsparungs-Analytics


```bash
rtk gain # Zusammenfassungsstatistiken
rtk gain --graph # ASCII-Graph (letzte 30 Tage)
rtk gain --history # Letzte Befehlshistorie
rtk gain --daily # Tag-für-Tag-Aufschlüsselung
rtk gain --all --format json # JSON-Export für Dashboards
rtk discover # Verpasste Einsparungsmöglichkeiten finden
rtk discover --all --since 7 # Alle Projekte, letzte 7 Tage
```

## Globale Flags


```bash
-u, --ultra-compact # ASCII-Icons, Inline-Format (zusätzliche Token-Einsparung)
-v, --verbose # Verbosität erhöhen (-v, -vv, -vvv)
```

## Beispiele

**Verzeichnisauflistung:**


```
# ls -la (45 Zeilen, ~800 Token)     # rtk ls (12 Zeilen, ~150 Token)
drwxr-xr-x 15 user staff 480 ... my-project/
-rw-r--r--  1 user staff 1234 ...   +-- src/ (8 Dateien)
...                                 |   +-- main.rs
                                    +-- Cargo.toml
```

**Git-Operationen:**


```
# git push (15 Zeilen, ~200 Token)   # rtk git push (1 Zeile, ~10 Token)
Enumerating objects: 5, done.        ok main
Counting objects: 100% (5/5), done.
Delta compression using up to 8 threads
...
```

**Test-Ausgabe:**


```
# cargo test (200+ Zeilen bei Fehler)  # rtk test cargo test (~20 Zeilen)
running 15 tests                       FAILED: 2/15 tests
test utils::test_parse ... ok          test_edge_case: assertion failed
test utils::test_format ... ok         test_overflow: panic at utils.rs:18
...
```

## Auto-Rewrite-Hook

Die effektivste Methode, rtk zu verwenden. Der Hook fängt Bash-Befehle transparent ab und schreibt sie vor der Ausführung zu rtk-Äquivalenten um.

**Ergebnis**: 100% rtk-Adoption über alle Konversationen und Subagenten, null Token-Overhead.

### Einrichtung


```bash
rtk init -g              # Hook installieren + RTK.md (empfohlen)
rtk init -g --auto-patch # Nicht-interaktiv (CI/CD)
rtk init -g --hook-only  # Nur Hook, kein RTK.md
rtk init --show          # Installation überprüfen
```

Nach Installation **Claude Code neu starten**.

### Umschriebene Befehle

| Roh-Befehl                                 | Umschrieben zu      |
| ------------------------------------------ | ------------------- |
| `git status/diff/log/add/commit/push/pull` | `rtk git ...`       |
| `gh pr/issue/run`                          | `rtk gh ...`        |
| `cargo test/build/clippy`                  | `rtk cargo ...`     |
| `cat/head/tail `                           | `rtk read `         |
| `rg/grep `                                 | `rtk grep `         |
| `ls`                                       | `rtk ls`            |
| `vitest/jest`                              | `rtk vitest run`    |
| `tsc`                                      | `rtk tsc`           |
| `eslint/biome`                             | `rtk lint`          |
| `prettier`                                 | `rtk prettier`      |
| `playwright`                               | `rtk playwright`    |
| `prisma`                                   | `rtk prisma`        |
| `ruff check/format`                        | `rtk ruff ...`      |
| `pytest`                                   | `rtk pytest`        |
| `pip list/install`                         | `rtk pip ...`       |
| `go test/build/vet`                        | `rtk go ...`        |
| `golangci-lint`                            | `rtk golangci-lint` |
| `docker ps/images/logs`                    | `rtk docker ...`    |
| `kubectl get/logs`                         | `rtk kubectl ...`   |
| `curl`                                     | `rtk curl`          |
| `pnpm list/outdated`                       | `rtk pnpm ...`      |

Befehle, die bereits `rtk` verwenden, Heredocs (`<<`) und nicht erkannte Befehle werden unverändert durchgereicht.

## Konfiguration

### Konfigurationsdatei

`~/.config/rtk/config.toml` (macOS: `~/Library/Application Support/rtk/config.toml`):


```toml
[tracking]
database_path = "/path/to/custom.db" # Standard: ~/.local/share/rtk/history.db

[hooks]
exclude_commands = ["curl", "playwright"] # Umschreibung für diese überspringen

[tee]
enabled = true  # Roh-Ausgabe bei Fehler speichern (Standard: true)
mode = "failures"  # "failures", "always" oder "never"
max_files = 20     # Rotationsgrenze
```

### Tee: Vollständige Ausgabewiederherstellung

Wenn ein Befehl fehlschlägt, speichert RTK die vollständige ungefilterte Ausgabe, damit die LLM sie ohne Neu-Ausführung lesen kann:


```
FAILED: 2/15 tests
[full output: ~/.local/share/rtk/tee/1707753600_cargo_test.log]
```

### Deinstallation


```bash
rtk init -g --uninstall  # Hook, RTK.md, settings.json-Eintrag entfernen
cargo uninstall rtk      # Binary entfernen
brew uninstall rtk       # Wenn über Homebrew installiert
```

## Dokumentation

- **[TROUBLESHOOTING.md](docs/TROUBLESHOOTING.md)** - Häufige Probleme beheben
- **[INSTALL.md](INSTALL.md)** - Detaillierter Installationsleitfaden
- **[ARCHITECTURE.md](ARCHITECTURE.md)** - Technische Architektur
- **[SECURITY.md](SECURITY.md)** - Sicherheitsrichtlinie und PR-Review-Prozess
- **[AUDIT_GUIDE.md](docs/AUDIT_GUIDE.md)** - Leitfaden für Token-Einsparungs-Analytics

## Mitwirken

Beiträge willkommen! Siehe das **[Contributing Guide](CONTRIBUTING.md)** für Benennung von Branches, PR-Prozess, Testanforderungen und Coding-Practices.

Treten Sie der Community auf [Discord](https://discord.gg/pvHdzAec) bei.

## Lizenz

MIT License - siehe [LICENSE](LICENSE) für Details.

## Kontakt

- Website: [https://www.rtk-ai.app](https://www.rtk-ai.app)
- E-Mail: [contact@rtk-ai.app](mailto:contact@rtk-ai.app)
- Issues: [https://github.com/rtk-ai/rtk/issues](https://github.com/rtk-ai/rtk/issues)