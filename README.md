# Kztunning-magisk-kernelSU-
ESTE MODULO ESTÁ EM BETA E FOI FEITO SOMENTE PARA O MEDIATEK HELIO G99 / MALIG57!!! (obs: por algum motivo quando você tentar ver o modulo no KernelSU do root temporário o app crasha. e se você quiser achar algum modulo que esta abaixo dele você vai ter q pesquisar o modulo especifico)

um modulo feito para extrair o maximo de perfomance do HelioG99 (novos processadores em breve)

# Kztuning MAX PERFORMANCE — Helio G99 / Mali-G57 MC2

> **Módulo Magisk/KernelSU** — Samsung Galaxy A24 (SM-A245M) — One UI 8.5  
> **Versão:** v8.5-G99-max-perf-no-notify | **Código:** 21  
> **100% Runtime** — Zero escritas em partição, sobrevive a soft-reboot, sem notificações.

---

## 📋 Visão Geral

Este módulo aplica tuning **máximo de performance** no Helio G99 (MT6789) com GPU Mali-G57 MC2. Tudo é aplicado em **runtime** via sysfs, props e settings — sem tocar `/vendor`, `/system` ou `boot.img`.

---

## ⚙️ O Que o Módulo Mexe (Lista Completa)

### 1. CPU — Governor & Frequências
| Parâmetro | Valor | Local |
|-----------|-------|-------|
| Governor (little A55 + big A76) | `performance` | `/sys/devices/system/cpu/cpufreq/policy*/scaling_governor` |
| Piso mínimo (little) | 50% → **1000 MHz** | `policy0/scaling_min_freq` |
| Piso mínimo (big) | 50% → **1100 MHz** | `policy6/scaling_min_freq` |
| Teto máximo | 100% (sem limite) | `policy*/scaling_max_freq = cpuinfo_max_freq` |
| Boost em toque | 90% por **200 ms** | via `input_booster` + `apply_floors` |
| Boost no app launch | 95% por **800 ms** | `cpu_boost_launch()` |
| `schedutil` tuning (se usado) | `up_rate=0`, `down_rate=50ms`, `rate_limit=1ms` | `policy*/schedutil/*` |

### 2. Scheduler (Kernel) — Apenas Parâmetros Desbloqueados no Samsung
| Parâmetro | Valor | Local |
|-----------|-------|-------|
| `sched_rr_timeslice_ms` | **1** | `/proc/sys/kernel/sched_rr_timeslice_ms` |
| `sched_child_runs_first` | **1** | `/proc/sys/kernel/sched_child_runs_first` |
| `sched_boost` | **1** (on launch) | `/proc/sys/kernel/sched_boost` |
| `printk` | `0 0 0 0` (silencia log kernel) | `/proc/sys/kernel/printk` |
| `perf_cpu_time_max_percent` | **5** | `/proc/sys/kernel/perf_cpu_time_max_percent` |
| `sched_schedstats` | **0** | `/proc/sys/kernel/sched_schedstats` |
| `nmi_watchdog` | **0** | `/proc/sys/kernel/nmi_watchdog` |

### 3. GPU Mali-G57 MC2 (via `mtk-dvfsrc-devfreq`)
| Parâmetro | Valor | Local |
|-----------|-------|-------|
| Governor | `performance` | `/sys/class/devfreq/mtk-dvfsrc-devfreq/governor` |
| Piso mínimo | 50% → **~2.1 GHz** | `min_freq` |
| Teto máximo | 100% → **4.266 GHz** | `max_freq` |
| Boost em toque | **200 ms** | `input_booster/boost_duration` |

### 4. GED — GPU Event Driver (Samsung/MediaTek)
| Parâmetro | Valor | Módulos Alvo |
|-----------|-------|--------------|
| `boost_enable` | **1** | `gpu`, `mali_kbase`, `mtk_dvfsrc` |
| `boost_level` | **3** (máximo) | mesmos |
| `dvfs_enable` | **1** | mesmos |
| `dvfs_margin` | **0** (freq exata) | mesmos |
| `thermal_enable` | **0** (desliga throttle GED) | mesmos |
| `boost_enable` (devfreq) | **1** | `mtk-dvfsrc-devfreq` |

> ⚠️ `GED_THERMAL_ENABLE=0` remove proteção térmica do driver GPU. Monitorado pelo **Thermal Guard** do módulo.

### 5. Hardware Acceleration (HWUI / Vulkan / Skia / EGL / GRALLOC / SurfaceFlinger)
#### HWUI / RenderThread / Vulkan
| Prop | Valor | Descrição |
|------|-------|-----------|
| `debug.hwui.renderer` | `skiavk` | Skia + Vulkan backend |
| `debug.hwui.skia_atlas_enabled` | `1` | Atlas de texturas Skia |
| `debug.hwui.use_vulkan` | `1` | Backend Vulkan |
| `debug.hwui.vulkan_async` | `1` | Submit assíncrono Vulkan |
| `debug.hwui.disable_vsync` | `1` | Desliga VSYNC no HWUI |
| `debug.hwui.triple_buffer` | `0` | Desliga triple buffer (latência) |
| `debug.hwui.profile` | `false` | Desliga profiler |
| `debug.hwui.overdraw` | `0` | Desliga debug overdraw |

#### HWUI Caches (MB)
| Prop | Valor |
|------|-------|
| `debug.hwui.shadow_cache_size` | 8 |
| `debug.hwui.layer_cache_size` | 48 |
| `debug.hwui.rro_cache_size` | 8 |
| `debug.hwui.text_cache_size` | 16 |
| `debug.hwui.path_cache_size` | 8 |
| `debug.hwui.buffer_age` | 3 |
| `debug.hwui.shape_cache_size` | 4 |
| `debug.hwui.gradient_cache_size` | 2 |
| `debug.hwui.dropdown_cache_size` | 2 |
| `debug.hwui.font_cache_size` | 4 |

#### EGL / Gralloc
| Prop | Valor |
|------|-------|
| `debug.egl.swap_behavior` | 1 (preserve buffer) |
| `debug.egl.force_msaa` | 0 |
| `debug.egl.profiler` | 0 |
| `debug.egl.trace` | 0 |
| `debug.gralloc.enable_ubwc` | 1 (compressão UBWC Samsung) |

#### SurfaceFlinger
| Prop | Valor |
|------|-------|
| `debug.sf.disable_vsync` | 1 |
| `debug.sf.latch_unsignaled` | 1 |
| `debug.sf.hwc` | 1 (HWC enabled) |
| `debug.sf.early_phase_offset_ns` | 500000 |
| `debug.sf.early_app_phase_offset_ns` | 500000 |
| `debug.sf.early_gl_phase_offset_ns` | 1500000 |
| `debug.sf.present_time_offset_ns` | 0 |
| `debug.sf.vsync_offset_ns` | 0 |
| `debug.sf.use_hwc_vsync` | 1 |

### 6. Display — 90 Hz Lock
| Setting | Valor |
|---------|-------|
| `min_refresh_rate` | `90.0` |
| `peak_refresh_rate` | `90.0` |
| `refresh_rate_mode` | `1` (fixo) |
| Reaplicação | A cada **5 s** (live.sh watchdog) |

### 7. Touch / Input Booster
| Parâmetro | Valor | Local |
|-----------|-------|-------|
| `enable_event` | 1 | `/sys/class/input_booster/enable_event` |
| Touch head/tail | 1 ms / 5 ms | `input_booster/touch/{head_time,tail_time}` |
| Multitouch head/tail | 1 ms / 5 ms | `input_booster/multitouch/{head_time,tail_time}` |
| Key head/tail | 1 ms / 5 ms | `input_booster/key/{head_time,tail_time}` |
| `debug_level` | 0 | `/sys/class/input_booster/debug_level` |
| Touch sampling rate | **240 Hz** | Painel DSI + Goodix (`touch_sampling_rate`, `sampling_rate`, `report_rate`) |
| CPU boost on touch | 90% / 200 ms | via `apply_floors` |
| GPU boost on touch | 200 ms | `input_booster/boost_duration` |

### 8. ZRAM — Compressão zstd 8 Streams
| Parâmetro | Valor | Local |
|-----------|-------|-------|
| Algoritmo | `zstd` | `/sys/block/zram0/comp_algorithm` |
| Streams | **8** | `/sys/block/zram0/max_comp_streams` |
| Tamanho | **Auto % RAM** (≤4.5GB=150%, ≤6.5GB=125%, >6.5GB=100%) | `/sys/block/zram0/disksize` |
| Dispositivo | `/dev/block/zram0` | swap ativo |

### 9. VM / RAM Otimização Agressiva
| Parâmetro | Valor | Local |
|-----------|-------|-------|
| `swappiness` | **180** | `/proc/sys/vm/swappiness` |
| `vfs_cache_pressure` | **300** | `/proc/sys/vm/vfs_cache_pressure` |
| `dirty_ratio` | **5** | `/proc/sys/vm/dirty_ratio` |
| `dirty_background_ratio` | **2** | `/proc/sys/vm/dirty_background_ratio` |
| `dirty_expire_centisecs` | **500** | `/proc/sys/vm/dirty_expire_centisecs` |
| `dirty_writeback_centisecs` | **500** | `/proc/sys/vm/dirty_writeback_centisecs` |
| `watermark_scale_factor` | **15** | `/proc/sys/vm/watermark_scale_factor` |
| `watermark_boost_factor` | **0** | `/proc/sys/vm/watermark_boost_factor` |
| `oom_dump_tasks` | **0** | `/proc/sys/vm/oom_dump_tasks` |
| `panic_on_oom` | **0** | `/proc/sys/vm/panic_on_oom` |
| `page-cluster` | **0** | `/proc/sys/vm/page-cluster` |
| `laptop_mode` | **0** | `/proc/sys/vm/laptop_mode` |
| `compaction_proactiveness` | **100** | `/proc/sys/vm/compaction_proactiveness` |
| `min_free_kbytes` | **3% RAM** (max 64 MB) | `/proc/sys/vm/min_free_kbytes` |
| THP | `never` (enabled + defrag) | `/sys/kernel/mm/transparent_hugepage/{enabled,defrag}` |
| KSM | **Desabilitado** (Samsung bloqueia) | `/sys/kernel/mm/ksm/run = 0` |
| zswap | **Desabilitado** | `/sys/module/zswap/parameters/enabled = N` |
| `vma_ra_enabled` | `false` | `/sys/kernel/mm/swap/vma_ra_enabled` |

#### Daemon de Limpeza (background)
- **Trigger:** PSI avg10 ≥ threshold **OU** RAM livre < `CLEAN_BELOW_PERCENT`
- **Ações:** `sync` + `drop_caches=3` + `compact_memory=1` + `am kill-all` + `kill_heavy_background`
- **Drop caches periódico:** pagecache only (mode 1) a cada **60 s**
- **Kill heavy bg:** apps > **200 MB** RSS (exceto system_server, zygote, surfaceflinger, init)

### 10. LMK / OOM
| Status | Descrição |
|--------|-----------|
| **DESABILITADO** (`LMK_TUNE=0`) | Samsung usa `lmkd` userspace; kernel module não funciona |

### 11. App Launch Boost
| Parâmetro | Valor |
|-----------|-------|
| CPU boost | 95% por **800 ms** |
| `sched_child_runs_first` | 1 |
| `sched_boost` | 1 |
| I/O priority foreground | `read_expire=100`, `write_expire=500`, `writes_starved=2`, `front_merges=1` |
| Zygote prefetch | `sys.zygote.prefetch=1` |
| Background cpuset | `0-5` (little cores only) |
| Background mem limit | 50% |

### 12. I/O Scheduler
| Dispositivo | Scheduler | Read-ahead | Extras |
|-------------|-----------|------------|--------|
| `mmcblk*`, `sd*` (físicos) | **bfq** | **512 KB** | `slice_idle=0` |
| `dm-*` (lógicos) | — (read-only) | **512 KB** | — |

### 13. Kernel Entropy
| Parâmetro | Valor |
|-----------|-------|
| `read_wakeup_threshold` | 64 |
| `write_wakeup_threshold` | 128 |
| `entropy_avail` | 4096 (tentativa) |

### 14. Thermal Guard (Proteção Inteligente)
| Gatilho | Ação |
|---------|------|
| Bateria ≥ **45°C** **OU** SoC ≥ **75°C** | Libera floors CPU/GPU (`release_floors`), para boost, loga |
| Bateria ≤ **42°C** **E** SoC ≤ **67°C** | Restaura floors (`apply_floors`), reativa boost |
| Poll | **1 s** |

### 15. CPU Hotplug
| Verificação | Esperado |
|-------------|----------|
| `online` == `present` | Todos núcleos online |

---

## 📁 Estrutura do Módulo
```
kztuning_mod/
├── module.prop          # Metadados (id, name, version, description, author)
├── service.sh           # Entry point KernelSU (boot_completed → optimize.sh)
├── optimize.sh          # Script principal (aplica tudo + daemon + live.sh)
├── lib.sh               # Funções de tuning (CPU/GPU/HWUI/GED/Touch/RAM/ZRAM/IO)
├── config.sh            # Todas as configs (Helio G99 / Mali-G57 MC2)
├── action.sh            # Status/check completo do módulo
├── live.sh              # Daemon persistente (Hz watchdog, thermal, reaplica floors/GED/HWUI)
├── kernel.sh            # Placeholder
└── module.log           # Log de execução (criado em runtime)
```

---

## 🔄 Fluxo de Execução
1. **Boot** → `service.sh` aguarda `sys.boot_completed=1` + 5s
2. **optimize.sh** roda em background:
   - Detecta Helio G99 (aborta se não for)
   - Aplica: CPU gov → Scheduler → I/O → GPU gov → GED → HW Accel → Touch Boost → ZRAM → VM/RAM → LMK → Entropy → Floors → cpuset bg
   - 90 Hz (2x com espera 5s)
   - Touch fixes + VSYNC + App Launch Boost
   - Limpeza inicial (`sync`, `drop_caches=3`, `compact_memory`, `am kill-all`, `fstrim`)
   - Inicia **daemon vras** (limpeza por pressão + drop_caches periódico + kill heavy bg)
   - Inicia **live.sh** (Hz watchdog + thermal guard + reaplica floors/GED/HWUI/Touch/VSYNC a cada 5s)
3. **action.sh** → relatório completo de status (use via terminal su)

---

## 📊 Como Verificar se Funcionou
```bash
# Via terminal root (su)
sh /data/adb/modules/kztuning_mod/action.sh
# ou
sh /data/adb/ksu/modules/kztuning_mod/action.sh
```

Saída esperada: tudo `[OK]` — CPU/GPU governors=performance, pisos aplicados, ZRAM zstd 8 streams, 90Hz confirmado, HWUI/Vulkan props ativos, Input Booster enabled, Thermal Guard monitorando, daemon vivo.

---

## ⚠️ Avisos & Compatibilidade
- **Apenas Helio G99 (MT6789)** — aborta em outro SoC.
- **One UI 8.5 / Android 15** — testado no Galaxy A24 SM-A245M.
- **KernelSU / Magisk** — `service.sh` compatível com ambos.
- **Sem notificações** — 100% silencioso.
- **Thermal Guard** protege contra superaquecimento (GED thermal desligado).
- **LMK desabilitado** — Samsung usa `lmkd` userspace.
- **KSM desabilitado** — Samsung bloqueia no kernel de produção.
- **Zero escritas em partição** — sobrevive a OTA / soft-reboot / disable module.

---

## 🛠 Personalização
Edite `config.sh` antes de empacotar/instalar:
- `REFRESH_HZ=0` → desliga 90 Hz lock
- `THERMAL_GUARD=0` → desliga proteção térmica
- `CPU_MIN_PCT=0` / `GPU_MIN_PCT=0` → remove pisos
- `TOUCH_FIX=0` → desliga touch fixes
- `HW_ACCEL=0` → desliga HW accel
- `GED_BOOST_ENABLE=0` → desliga GED boost
- `VM_SWAPPINESS=100` → menos agressivo (padrão Android)

---

## 📦 Instalação
1. Baixe `kztuning_v8.5-G99-max-perf.zip`
2. Magisk/KernelSU → **Módulos** → **Instalar do armazenamento** → selecione o ZIP
3. **Reinicie**
4. Verifique: `su -c sh /data/adb/modules/kztuning_mod/action.sh`

---

## 🔧 Desinstalação / Reset
- Magisk/KernelSU → **Módulos** → **Remover** → Reiniciar
- Ou: `touch /data/adb/modules/kztuning_mod/remove` → Reiniciar
- **Nenhum rastro** em partições — tudo era runtime.

---

## 📝 Changelog Resumido
| Versão | Mudanças |
|--------|----------|
| **v8.5** | MAX PERFORMANCE: CPU/GPU governors=performance, pisos 50%, GED boost level 3 thermal=0, HWUI/Vulkan/Skia/EGL/GRALLOC/SF completo, 90Hz lock, Touch 240Hz + input_booster 1/5ms, ZRAM zstd 8 streams auto%, VM agressivo (swappiness=180), App Launch Boost 95%/800ms, Thermal Guard 45/75°C, Daemon PSI-based + drop_caches + kill heavy bg, live.sh watchdog 5s reaplica tudo. |
| v8.x | Versões anteriores (RAM opt, ZRAM, CPU gov, etc.) |

---

## 🧪 Testado Em
- **Dispositivo:** Samsung Galaxy A24 4G (SM-A245M)
- **SoC:** MediaTek Helio G99 (MT6789)
- **GPU:** Mali-G57 MC2
- **ROM:** One UI 8.5 (Android 15)
- **Root:** KernelSU / Magisk Delta

---

## ⚖️ Licença / Créditos
- Base: **Kztuning** (original)
- Modificações: Tuning máximo para G99/G57 + One UI 8.5
- **Uso por sua conta e risco.** Thermal Guard mitiga, mas `GED_THERMAL_ENABLE=0` remove proteção do driver GPU. Monitore temperaturas.

---
Desenvolvido por @Kzrxe7z / @nxt_nix2
