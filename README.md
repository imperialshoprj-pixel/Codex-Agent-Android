# Codex Agent — Agente de Engenharia Autônomo para Android

App Android (Kotlin + Jetpack Compose) estilo OpenAI Codex / OpenCode / Claude Code, rodando
100% nativo no dispositivo: agente de IA com ferramentas de shell, arquivos, web, APK reverse
engineering, checkpoints Git, terminal live e compilação — sem pedidos manuais de permissão
por passo (modo YOLO).

## Stack fixada

| Item | Versão |
|---|---|
| Kotlin | 2.0.21 |
| AGP | 8.7.2 |
| KSP | 2.0.21-1.0.27 |
| Compose BOM | 2024.10.01 |
| Room | 2.6.1 |
| OkHttp (+sse) | 4.12.0 |
| Kotlinx Serialization | 1.7.3 |
| Coil | 2.7.0 |
| Shizuku | 13.1.5 |
| compileSdk / minSdk / targetSdk | 35 / 24 / **28** |

`targetSdk = 28` é obrigatório: contorna a restrição W^X do SELinux no Android 10+, permitindo
`execve` de binários baixados para `filesDir/usr`. App destinado a sideload (Google Play exigiria
targetSdk maior e não permitiria `MANAGE_EXTERNAL_STORAGE` sem justificativa).

## Árvore completa do projeto

```
CodexAgent/
├── README.md                                   [P1] ✅
├── settings.gradle.kts                         [P1] ✅
├── build.gradle.kts                            [P1] ✅
├── gradle.properties                           [P1] ✅
├── .gitignore                                  [P1] ✅
├── gradle/
│   └── libs.versions.toml                      [P1] ✅
└── app/
    ├── build.gradle.kts                        [P1] ✅
    ├── proguard-rules.pro                      [P1] ✅
    ├── schemas/                                (gerado pelo KSP/Room)
    └── src/main/
        ├── AndroidManifest.xml                 [P1] ✅
        ├── res/
        │   ├── drawable/ic_app_logo.xml        [P1] ✅
        │   ├── values/strings.xml              [P1] ✅
        │   ├── values/colors.xml               [P1] ✅
        │   ├── values/themes.xml               [P1] ✅
        │   └── xml/
        │       ├── network_security_config.xml [P1] ✅
        │       ├── file_paths.xml              [P1] ✅
        │       ├── backup_rules.xml            [P1] ✅
        │       └── data_extraction_rules.xml   [P1] ✅
        └── java/com/codex/agent/
            ├── data/
            │   ├── model/
            │   │   ├── Enums.kt                [P1] ✅
            │   │   └── Constants.kt            [P1] ✅
            │   ├── db/
            │   │   ├── AppDatabase.kt          [P1] ✅
            │   │   └── Converters.kt           [P1] ✅
            │   ├── entity/
            │   │   ├── ProviderConfigEntity.kt [P1] ✅
            │   │   ├── WorkspaceEntity.kt      [P1] ✅
            │   │   ├── AgentSessionEntity.kt   [P1] ✅
            │   │   ├── ChatMessageEntity.kt    [P1] ✅
            │   │   ├── CheckpointEntity.kt     [P1] ✅
            │   │   ├── BackgroundJobEntity.kt  [P1] ✅
            │   │   └── SettingEntity.kt        [P1] ✅
            │   ├── dao/
            │   │   ├── ProviderConfigDao.kt    [P1] ✅
            │   │   ├── WorkspaceDao.kt         [P1] ✅
            │   │   ├── AgentSessionDao.kt      [P1] ✅
            │   │   ├── ChatMessageDao.kt       [P1] ✅
            │   │   ├── CheckpointDao.kt        [P1] ✅
            │   │   ├── BackgroundJobDao.kt     [P1] ✅
            │   │   └── SettingsDao.kt          [P1] ✅
            │   └── repository/
            │       ├── SettingsRepository.kt   [P1] ✅
            │       ├── ProviderRepository.kt   [P1] ✅
            │       └── WorkspaceRepository.kt  [P1] ✅
            ├── llm/
            │   ├── LlmModels.kt                [P2] ✅ DTOs unificados + TokenEstimator
            │   ├── LlmClient.kt                [P2] ✅ Multi-API + dual-model + SSE + vision + retry
            │   └── AnthropicAdapter.kt         [P2] ✅ (embutido em LlmClient — protocolo /v1/messages)
            ├── shell/
            │   ├── ShellExecutor.kt            [P2] ✅ Termux Bridge + ProcessBuilder + bg manager + truncator
            │   └── EnvironmentSetup.kt         [P2] ✅ Toolchain dev/APK + keystore + bootstrap interno
            ├── agent/
            │   ├── ToolDefinitions.kt          [P3] ✅ Schemas das 10 ferramentas + custom tools
            │   ├── ToolExecutor.kt             [P3] ✅ Execução (paralela) + git checkpoints + Shizuku
            │   ├── ApkProjectTemplates.kt      [P5] ✅ Gerador de projetos (Java+XML / Kotlin+Compose)
            │   ├── AndroidBuildTool.kt         [P5] ✅ scaffold + build direto (aapt2/javac/d8/apksigner)
            │   └── AgentOrchestrator.kt        [P3] ✅ Loop + streaming + compaction + anti-loop + AGENT.md
            ├── service/
            │   └── AgentForegroundService.kt   [P4] ✅ Notificação persistente + wake lock + Stop
            ├── CodexAgentApplication.kt        [P4] ✅
            └── ui/
                ├── Theme.kt                    [P4] ✅ Dark IDE theme
                ├── MainActivity.kt             [P4] ✅ DI + navegação (4 abas)
                ├── PermissionCheckBanner.kt    [P4] ✅ Setup wizard (Termux/Shizuku/storage/toolchain)
                ├── AgentChatScreen.kt          [P4] ✅ Chat + streaming + tool cards + vision
                ├── TerminalScreen.kt           [P4] ✅ Terminal live + process manager
                ├── FileExplorerScreen.kt       [P4] ✅ Árvore + editor + APK Inspector
                └── ApiSettingsScreen.kt        [P4] ✅ Provedores, dual-model, system prompt
```

## Parte 5 — Engenharia de APKs e criação de apps no dispositivo

Templates: java_xml (compila no celular), kotlin_compose (Android Studio), iptv (ExoPlayer+M3U), webview, game (SurfaceView), service (ForegroundService).
Ações: clone_apk (clonagem 1:1 com verificação SHA-256), compare_apks (diff de dois APKs), rebuild_and_sign.

## Parte 6 — ToolHub: ferramentas avançadas de RE (28 receitas verificadas)

Catálogo: apkeditor, apktool, jadx, smali, baksmali, dex2jar, cfr, procyon, bytecode-viewer, recaf, uber-apk-signer, bundletool, revanced-cli+patches, apk-mitm, enjarify, androguard, apkleaks, frida-tools, frida-server (root), objection, android-tools (adb), radare2, lief, il2cppdumper (Unity), ghidra-headless (pesada), mobsf (experimental).

Descoberta universal: search_github → list_release_assets → install_custom (qualquer URL + pós-comando + wrapper no PATH).

## Parte 7 — Segurança (OWASP)

- SecretsStore (EncryptedSharedPreferences / Android Keystore) para API keys
- Trava de segurança real contra comandos destrutivos
- Path traversal guard em file_manager
- network_security_config restrito a HTTPS (cleartext apenas localhost)
- AuditLog no Room (rastreabilidade de comandos)
- BiometricLockScreen opcional (USE_BIOMETRIC)
- URL normalization automática (/v1) + resolvePreferred prefere providers com key

## Parte 8 — RAG e busca semântica ✅

- `CodebaseIndex` — indexador TF-IDF 100% offline (sem embeddings externos).
- Injeção automática dos top-5 arquivos relevantes no system prompt antes de cada turno.
- Ferramenta `search_semantic(query, top_k)` para buscas adicionais.
- Ignora `.git`, `.codex_git`, `build`, `node_modules`, etc.; stop words filtradas.

## Parte 9 — Análise estática de APK ✅ (auditoria do próprio app)

- `apk_security_audit(apk_path)` — detecta componentes exportados sem permissão, `debuggable`,
  `allowBackup`, cleartext traffic, secrets hardcoded (Google/OpenAI/GitHub/AWS keys),
  WebView inseguro (JS + `addJavascriptInterface`), SQL injection em ContentProviders e libs `.so`.
- Relatório estruturado por severidade (CRITICAL/HIGH/MEDIUM/LOW/INFO).

## Parte 10 — UI polish ✅

- Syntax highlighting no visualizador de código (Kotlin/Java/XML/Smali/JSON/Python/Shell) via
  `SyntaxHighlighter` (AnnotatedString, sem dependências externas).
- Diff colorido (verde=adição, vermelho=remoção, ciano=cabeçalho).
- ANSI colors no terminal: `AnsiParser` converte escape sequences (`\033[31m`) em `AnnotatedString`.

## Parte 11 — MCP client ✅

- `mcp_client` sobre Streamable HTTP / JSON-RPC 2.0 (protocolo MCP 2024-11-05).
- Ações: `add_server`, `list_servers`, `remove_server`, `list_tools`, `call_tool`, `ping`.
- Configurações em `.codex/mcp_servers.json`; headers customizados suportados.

## Parte 12 — Git real + PR automation ✅

- `git_manager` — status, log, branches, create_branch, checkout, diff, add_commit, stash/pop,
  merge, set_remote, push, pull.
- `create_pr` via GitHub REST API; token salvo no Android Keystore (`set_token`), nunca em texto puro.

## Parte 13 — Sub-agentes ✅

- `delegate_task(task, max_steps)` — agente com contexto isolado e ferramentas de leitura,
  loop limitado e relatório técnico conciso. Permite paralelizar investigações.

## Parte 14 — Testes ✅

- 17 testes (JUnit4 + MockWebServer): `ShellSafetyGuardTest` (12), `LlmClientTest` (3),
  `CodebaseIndexTest` (2). `./gradlew :app:testDebugUnitTest` — todos passando.

## Parte 15 — Observabilidade ✅

- Crash reporting local + métricas de uso de tools (duração/sucesso), 100% on-device.
- `observability(action='report'|'enable'|'disable')` exporta relatório; nada é enviado a servidores.

## Parte 16 — Integrações de APIs ✅

- Provedores seed: OpenAI, Anthropic (nativo), Gemini, DeepSeek, OpenRouter, Groq, Ollama (local)
  e **Vyce AI** (`https://vyceai.com/v1`).
- Vyce AI: URL normalization `/v1`, fallback de key `vyce_free_access`, descoberta de modelos via `/models`.

Nova ferramenta `android_build_tool` (10ª ferramenta do agente):

- `scaffold_project` — gera um projeto Android Studio completo:
  - **`java_xml`**: app Java puro (sem androidx) compilável **no próprio celular** via pipeline direto;
  - **`kotlin_compose`**: app Kotlin + Jetpack Compose moderno (Gradle/BOM 2024.10.01) para abrir no Android Studio;
  - Parâmetros: nome, package, minSdk/targetSdk, versionCode/Name, permissões.
- `provision_android_sdk` — instala `aapt2/aapt/d8/apksigner/zipalign` no Termux e baixa o
  **android.jar oficial** (platform-34, ~60MB, `dl.google.com`) para `$HOME/android-sdk`.
- `build_apk` — pipeline direto no dispositivo, sem Gradle:
  `aapt2 compile+link (recursos + R.java) → javac (contra android.jar) → d8 (dex) → zip → zipalign -p 4 → apksigner (debug.keystore automática)` → `app-debug.apk` pronto para `android_system_tool(action='install_apk')`.
  Suporta layout Gradle (`app/src/main`) e flat; injeta `package=` no manifest quando o projeto usa `namespace`.
- `create_keystore` — keystrokes de assinatura sob demanda (alias/senha/DN).
- `gradle_build` — roda `./gradlew` quando o projeto tem wrapper + SDK completo.

O `DEFAULT_SYSTEM_PROMPT` recebeu as regras 9–11 com o fluxo completo
(provisionar → scaffold → editar com patch_file → build → instalar → logcat em falhas).

## Parte 6 — ToolHub: ferramentas avançadas de RE do GitHub (reais e verificadas)

11ª ferramenta `tool_hub` — instalador/executador universal com **verificação real pós-instalação**
(executa a ferramenta e mostra a saída; nada é dado como "instalado" sem prova):

| ID | Ferramenta | Fonte real | Notas |
|---|---|---|---|
| `apkeditor` | **APKEditor (REAndroid)** | `REAndroid/APKEditor` releases (jar runnable) | mescla xapk/apks, edita recursos |
| `apktool` / `jadx` | Apktool / jadx | `iBotPeaches/Apktool`, `skylot/jadx` | jadx zip normalizado p/ `{TOOL_DIR}/jadx` |
| `smali` / `baksmali` | smali/baksmali | `google/smali` | maior asset = jar runnable |
| `dex2jar` | dex2jar | `pxb1988/dex2jar` (`dex-tools-*.zip`) | scripts d2j-* |
| `uber-apk-signer` | uber-apk-signer | `patrickfav/uber-apk-signer` | assina v1+v2+v3 e alinha |
| `bundletool` | bundletool | `google/bundletool` | aab → apks |
| `revanced-cli` / `revanced-patches` | ReVanced | `ReVanced/revanced-*` | patches publicam **.rvp** (padrão cobre jar+rvp) |
| `apk-mitm` | apk-mitm | npm `apk-mitm` | patch p/ interceptar HTTPS |
| `enjarify` | enjarify | `Google/enjarify` (git clone) | `PYTHONPATH` + `python3 -m enjarify` |
| `androguard` / `apkleaks` / `frida-tools` / `objection` | pip | PyPI | verificação via import Python |
| `frida-server` | frida-server arm64 | `frida/frida` releases (`*-android-arm64.xz`) | instala `xz-utils`, requer root |
| `android-tools` | adb/fastboot | Termux `android-tools` | Shizuku wireless / adb |
| `radare2` | radare2 | Termux `radare2` | análise de .so |
| `ghidra-headless` | Ghidra headless | `NationalSecurityAgency/ghidra` | zip normalizado p/ `{TOOL_DIR}/ghidra`; PESADA |
| `mobsf` | MobSF | git clone + requirements | EXPERIMENTAL no Termux |

Dinâmico e universal: `search_github` (API de busca), `list_release_assets` (assets reais do
latest release — o agente nunca inventa URL), `install_custom` (qualquer URL + pós-comando +
wrapper no PATH). Estado em `~/.local/share/codex_tools/installed.json` + registro em
`.codex/dependencies.md`. Binários vivem em armazenamento executável (`$HOME`), nunca em `/sdcard`.

## Esquema do banco (Room v1)

- `provider_configs` — provedores LLM; `mainModel` (Coder/Architect) + `utilityModel` (economia).
- `workspaces` — raízes de projeto; um ativo por vez (`/sdcard/CodexProjects` padrão).
- `agent_sessions` — modo YOLO/PLAN_FIRST, status, `compactionSummary`, `lastCompactedMessageId`,
  `planText`, `totalTokensUsed`, `maxSteps`, override de `contextWindowTokens`.
- `chat_messages` — histórico completo com `toolCallId`, `reasoning`, `imageBase64` (visão),
  `SYSTEM_INJECTION` (anti-loop/compaction). Compaction apaga `id <= lastCompactedMessageId`.
- `checkpoints` — shadow commits Git para rollback com 1 toque (AUTO_SHADOW / MANUAL / AGENT).
- `background_jobs` — processos `is_background=true` (PID, logPath, status, Kill no Terminal).
- `settings` — chave-valor tipado (modo de execução, prompt base, limites, provider ativo).

## Como compilar

1. Android Studio Ladybug (ou superior, AGP 8.7.x) com JDK 17.
2. Abra a pasta `CodexAgent`; aceite o Gradle Sync (o wrapper pode ser gerado pelo próprio
   Studio na abertura, ou via `gradle wrapper --gradle-version 8.9`).
3. `Build > Make Project`.

## Notas de segurança (OWASP)

- `usesCleartextTraffic` + user certs habilitados são aceitáveis apenas porque este é um app de
  engenharia local; qualquer tráfego de API sensível deve usar HTTPS.
- API keys ficam apenas no Room local do dispositivo (sem backup do arquivo de banco para
  nuvem além do Android padrão; exportSchema em `app/schemas`).
- Modo YOLO executa comandos arbitrários no ambiente do usuário — a única trava inegociável é
  contra destruição do armazenamento (`rm -rf /`, wipe de `/sdcard`).
