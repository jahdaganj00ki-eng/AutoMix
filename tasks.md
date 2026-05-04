
# Tasks – AutoDJ Mix

## Phase 1: Solution-Gerüst und Projekte

- [ ] 1.1 Solution-Datei `AutoDjMix.sln` anlegen
- [ ] 1.2 Projekt `AutoDjMix.Domain` anlegen (.NET 8 classlib, keine externen Abhängigkeiten)
- [ ] 1.3 Projekt `AutoDjMix.Application` anlegen (.NET 8 classlib, Referenz auf Domain)
- [ ] 1.4 Projekt `AutoDjMix.Analysis` anlegen (.NET 8 classlib, Referenz auf Domain; NuGet: NAudio)
- [ ] 1.5 Projekt `AutoDjMix.Mixing` anlegen (.NET 8 classlib, Referenz auf Domain, Application)
- [ ] 1.6 Projekt `AutoDjMix.Transition` anlegen (.NET 8 classlib, Referenz auf Domain, Mixing)
- [ ] 1.7 Projekt `AutoDjMix.Rendering` anlegen (.NET 8 classlib, Referenz auf Domain, Transition; NuGet: NAudio)
- [ ] 1.8 Projekt `AutoDjMix.Persistence` anlegen (.NET 8 classlib, Referenz auf Domain; NuGet: Dapper, Microsoft.Data.Sqlite)
- [ ] 1.9 Projekt `AutoDjMix.Infrastructure` anlegen (.NET 8 classlib, Referenz auf Domain; NuGet: Serilog, Serilog.Sinks.File)
- [ ] 1.10 Projekt `AutoDjMix.App` anlegen (WPF .NET 8, Referenz auf alle Projekte; NuGet: Microsoft.Extensions.Hosting)
- [ ] 1.11 Projekt `AutoDjMix.Tests.Domain` anlegen (xUnit, Referenz auf Domain, Mixing; NuGet: xUnit, FluentAssertions, FsCheck.Xunit)
- [ ] 1.12 Projekt `AutoDjMix.Tests.Application` anlegen (xUnit, Referenz auf Application; NuGet: xUnit, Moq, FluentAssertions)
- [ ] 1.13 Projekt `AutoDjMix.Tests.Integration` anlegen (xUnit, Referenz auf Persistence, Application; NuGet: xUnit, Microsoft.Data.Sqlite)
- [ ] 1.14 Projekt `AutoDjMix.Tests.EndToEnd` anlegen (xUnit, Referenz auf App; NuGet: xUnit)

## Phase 2: Domain-Modelle

- [ ] 2.1 Enums anlegen: `TrackRole` (FOUNDATION, HYPNOTIC_DRIVER, PERCUSSIVE_BRIDGE, ATMOSPHERIC_LIFT, MELODIC_OPEN, PSY_EDGE, PEAK_TOOL, RESET_TOOL, NEUTRAL_TOOL), `TransitionType` (alle 11), `SetPhase` (WARMUP, HYPNOSIS, LIFT, PEAK, RESET), `AnalysisState` (Pending, Analyzing, Done, Failed), `DecisionCategory` (PERFECT, GOOD, RISKY, AVOID)
- [ ] 2.2 `TrackFeatures`-Klasse anlegen: alle 22 Features aus mixEngine.json mit korrekten Wertebereichen (energy [1-10], break_count [0-20], alle anderen [0-100])
- [ ] 2.3 `TrackSegments`-Klasse anlegen: IntroStart/End, OutroStart/End, Beats, Downbeats, PhraseBoundaries, Breaks, Builds, Drops, Hooks (alle als Zeitstempel-Listen), PeakData
- [ ] 2.4 `Track`-Klasse anlegen: TrackId, FilePath, FileName, FileHash (SHA-256), FileSizeBytes, Duration, AnalysisState, Features, Segments, RoleAuto, RoleManual, RoleLocked, WarnFlags, EffectiveRole-Property
- [ ] 2.5 Automation-Klassen anlegen: `AutomationPoint`, `EqAutomationProfile` (Low/Mid/High-Kurven), `FilterAutomationProfile` (HP/LP), `LoudnessAutomationProfile` (MaxLufsDelta=2.0)
- [ ] 2.6 `RiskFlag`-Record anlegen: RuleId, Description, Penalty
- [ ] 2.7 `TransitionPlan`-Klasse anlegen mit allen 15 Schema-Feldern aus mixEngine.json: FromTrackId, ToTrackId, FromRole, ToRole, TransitionType, TransitionStartInTrackASeconds, TransitionStartInTrackBSeconds, TransitionDurationBeats, TransitionDurationSeconds, LoopSource, LoopBars, BassSwapAtBeatOffset, EqAutomationProfile, FilterAutomationProfile, LoudnessAutomationProfile – plus interne Felder: TransitionId, MixPlanId, HardOverridesApplied, RiskFlags, FinalScore
- [ ] 2.8 `MixPlanEntry`-Klasse anlegen: EntryId, MixPlanId, Position, TrackId, IsBridgeTrack, PositionLocked, IsStartTrack, IsEndTrack
- [ ] 2.9 `MixPlan`-Klasse anlegen: MixPlanId, ProjectId, CreatedAt, IsStale, Tracks (List<MixPlanEntry>), Transitions (List<TransitionPlan>), TotalDuration, PhaseAssignments
- [ ] 2.10 `Project`-Klasse anlegen: ProjectId, Name, CreatedAt, UpdatedAt, CurrentMixPlanId, TrackIds
- [ ] 2.11 `RoleAssignment`-Klasse anlegen: TrackId, AutoRole, ManualRole, IsLocked, WeakHintBiasPoints (max 6)
- [ ] 2.12 Repository-Interfaces anlegen: `ITrackRepository`, `IMixPlanRepository`, `IProjectRepository`, `ISettingsRepository`
- [ ] 2.13 Service-Interfaces anlegen: `IJobService` (StartJobAsync, CancelCurrentJob, IsRunning), `JobProgress`-Record
- [ ] 2.14 Audio-Interfaces anlegen: `IAudioAnalyzer`, `IAudioDecoder`, `IBpmDetector`, `IKeyDetector`
- [ ] 2.15 Mixing-Interfaces anlegen: `IMixPlanner`, `IPairRuleEngine`, `IHardOverrideEngine`, `IRiskModel`, `IScoreCalculator`
- [ ] 2.16 Transition-Interface anlegen: `IDynamicTransitionEngine`, `IAutomationCurveCalculator`
- [ ] 2.17 Rendering-Interfaces anlegen: `IRenderingPipeline`, `ITimeStretcher`, `IFfmpegWrapper`

## Phase 3: MixEngine-Parser und Validator

- [ ] 3.1 `MixEngineConfig`-Klasse und alle Unterklassen anlegen: `WeightsConfig` (alle 11 Gewichte exakt aus mixEngine.json), `DecisionThresholds` (play_direct=82, play_with_constraints=68, bridge_required=54, avoid=53), `PairRule`, `GlobalRule`, `HardOverrideRule`, `RiskPenaltyRule`, `PhaseConfig`, `RoleDefinition`, `ExportPolicy`, `TransitionTypeConfig`
- [ ] 3.2 `mixEngine.json` als embedded resource in `AutoDjMix.Mixing` einbetten (Build Action: EmbeddedResource)
- [ ] 3.3 `MixEngineConfigLoader` implementieren: lädt embedded resource, deserialisiert via `System.Text.Json` mit `PropertyNameCaseInsensitive=true`, gibt `MixEngineConfig` zurück
- [ ] 3.4 `MixEngineConfigValidator` implementieren: prüft Vollständigkeit (26 Pair-Regeln, 9 Rollen, 11 Übergangstypen, 25 globale Regeln, 6 Hard Overrides, 7 Risk-Penalty-Regeln); wirft `InvalidOperationException` mit Detailmeldung bei Fehler
- [ ] 3.5 Unit-Tests für `MixEngineConfigValidator`: Test_ValidConfig_PassesValidation, Test_MissingPairRule_ThrowsException, Test_WrongRoleCount_ThrowsException, Test_AllTransitionTypesPresent

## Phase 4: SQLite-Persistenz und Migrationen

- [ ] 4.1 `DatabaseConnectionFactory` implementieren: erstellt/öffnet SQLite-Datenbank unter `%APPDATA%\AutoDjMix\autodj.db`, stellt Dapper-Connection bereit
- [ ] 4.2 Migrations-SQL-Skript `001_InitialSchema.sql` anlegen mit allen 11 CREATE TABLE Statements: Tracks, TrackFeatures, TrackSegments, Projects, MixPlans, MixPlanEntries, TransitionPlans, Locks, Settings, ExportHistory, SchemaVersion (exakt wie in design.md spezifiziert)
- [ ] 4.3 `DatabaseMigrationService` implementieren: liest SchemaVersion, führt ausstehende embedded SQL-Skripte sequenziell aus, schreibt neue Version
- [ ] 4.4 `TrackRepository` implementieren: GetByIdAsync, GetByHashAsync, GetAllAsync, SaveAsync (INSERT OR REPLACE), DeleteAsync mit CASCADE
- [ ] 4.5 `TrackFeaturesRepository` implementieren: SaveAsync, GetByTrackIdAsync (alle 22 Feature-Spalten)
- [ ] 4.6 `TrackSegmentsRepository` implementieren: SaveAsync, GetByTrackIdAsync (JSON-Serialisierung für Listen-Felder)
- [ ] 4.7 `MixPlanRepository` implementieren: GetByIdAsync, GetCurrentForProjectAsync, SaveAsync, MarkStaleAsync
- [ ] 4.8 `MixPlanEntryRepository` implementieren: SaveAllAsync, GetByMixPlanIdAsync (geordnet nach Position)
- [ ] 4.9 `TransitionPlanRepository` implementieren: SaveAllAsync, GetByMixPlanIdAsync (JSON-Serialisierung für Automation-Profile)
- [ ] 4.10 `ProjectRepository` implementieren: GetByIdAsync, GetAllAsync, SaveAsync, DeleteAsync
- [ ] 4.11 `LockRepository` implementieren: SaveAsync, GetByProjectIdAsync, DeleteAsync
- [ ] 4.12 `SettingsRepository` implementieren: GetAsync, SetAsync (Key-Value-Store)
- [ ] 4.13 `ExportHistoryRepository` implementieren: SaveAsync, GetAllAsync
- [ ] 4.14 Integrationstests für Persistenz: Test_TrackRepository_SaveAndLoad_RoundTrip, Test_MigrationService_RunsMigrationsInOrder, Test_TransitionPlanRepository_AllFieldsPersisted

## Phase 5: Import-Service

- [ ] 5.1 `ImportService` implementieren: nimmt Liste von Dateipfaden entgegen, prüft Dateiendung (mp3/m4a/wav), berechnet SHA-256-Hash, prüft auf Duplikate via `ITrackRepository.GetByHashAsync`, legt neue `Track`-Einträge an
- [ ] 5.2 Rekursiven Ordner-Scanner implementieren: durchsucht Verzeichnis und alle Unterverzeichnisse nach mp3/m4a/wav-Dateien
- [ ] 5.3 Deduplizierungs-Logik implementieren: Vergleich via absolutem Pfad UND SHA-256-Hash; zählt übersprungene Duplikate
- [ ] 5.4 Import-Hintergrundjob implementieren: läuft via `IJobService.StartJobAsync`, nutzt `SemaphoreSlim(4)` für max. 4 parallele Datei-Lesevorgänge, meldet Fortschritt „Importiere Datei X/Y – [Dateiname]", unterstützt `CancellationToken`
- [ ] 5.5 Fehlerbehandlung im Import: nicht lesbare Dateien überspringen, Fehler protokollieren, Import fortsetzen
- [ ] 5.6 `FFmpegWrapper.DecodeToPcmAsync` implementieren: startet ffmpeg-Prozess mit stdout-Pipe, liest PCM-Daten, behandelt Exit-Code ≠ 0 als Exception
- [ ] 5.7 `FFmpegWrapper.IsAvailable` implementieren: prüft ob ffmpeg.exe am konfigurierten Pfad vorhanden und ausführbar ist
- [ ] 5.8 Unit-Tests für ImportService: Test_DuplicateFiles_AreSkipped, Test_NonAudioFiles_AreIgnored, Test_RecursiveFolderScan_FindsAllFiles, Test_CancellationToken_StopsImport

## Phase 6: Audio-Analyse-Pipeline

- [ ] 6.1 `AudioDecoder` implementieren: NAudio primär (WaveFileReader/Mp3FileReader), FFmpeg-Fallback für .m4a und bei NAudio-Codec-Fehler; gibt `ISampleProvider` zurück
- [ ] 6.2 `Resampler` implementieren: konvertiert auf 44100 Hz Stereo; Mono-zu-Stereo-Upmix
- [ ] 6.3 `BpmDetector` implementieren: Onset-Detection via Spectral Flux + Autocorrelation im Bereich 60–200 BPM; Genauigkeit ±0,1 BPM; chunk-basiert (4096 Samples); berechnet `beat_confidence`
- [ ] 6.4 `BeatgridBuilder` implementieren: berechnet Beat-Timestamps, Downbeats, Phrasengrenzen (8/16/32 Beats) aus BPM-Ergebnis
- [ ] 6.5 `KeyDetector` implementieren: 12-Bin-Chromagramm → Krumhansl-Schmuckler-Profil-Matching → Camelot-Notation; berechnet `key_confidence`
- [ ] 6.6 `EnergyFeatureExtractor` implementieren: RMS → `energy` [1-10]; Spectral Centroid → `brightness`, `darkness`
- [ ] 6.7 `BassFeatureExtractor` implementieren: Sub-Bass RMS (20–250 Hz) → `bass_weight`
- [ ] 6.8 `GrooveFeatureExtractor` implementieren: Onset-Density → `groove_density`, `percussive_focus`
- [ ] 6.9 `MelodicFeatureExtractor` implementieren: Pitch-Salience → `melodic_presence`, `hook_dominance`, `psy_intensity`, `atmospheric_depth`
- [ ] 6.10 `StructureAnalyzer` implementieren: Novelty-Curve via Self-Similarity-Matrix → Segmentgrenzen → Klassifikation als Intro/Outro/Break/Build/Drop/Hook; berechnet `intro_utility`, `outro_utility`, `break_size`, `break_count`, `drop_impact`, `transition_flexibility`; berechnet `phrase_confidence`, `structure_confidence`
- [ ] 6.11 `ConfidenceCalculator` implementieren: aggregiert `beat_confidence`, `key_confidence`, `phrase_confidence`, `structure_confidence`
- [ ] 6.12 `FeatureClipper` implementieren: clippt alle Feature-Werte auf mixEngine.json-Bereiche (energy [1-10], break_count [0-20], alle anderen [0-100])
- [ ] 6.13 `RoleAssigner` implementieren: Energie-Range als primäres Kriterium (exakt aus mixEngine.json), schwache Hints max. 6 Bias-Punkte aus Tags/Ordnernamen/Dateinamen, sekundäre Feature-Kriterien, Fallback NEUTRAL_TOOL
- [ ] 6.14 `PeakDataGenerator` implementieren: downsampled Peak-Daten für Waveform-Rendering, speichert in `TrackSegments.PeakData`
- [ ] 6.15 `AudioAnalyzer` implementieren: orchestriert alle 15 Pipeline-Schritte, meldet Fortschritt „Analysiere Track X/Y – [Schritt-Name]", unterstützt `CancellationToken`, schreibt Ergebnis in SQLite-Cache
- [ ] 6.16 `AnalysisService` implementieren: verwaltet Analyse-Queue, nutzt `SemaphoreSlim(2)` für max. 2 parallele Analyse-Jobs, prüft Cache-Gültigkeit (SHA-256-Hash + Änderungsdatum)
- [ ] 6.17 Unit-Tests Analyse: Test_BpmDetector_SyntheticAudio_AccuracyWithin01, Test_KeyDetector_KnownKey_ReturnsCorrectCamelot, Test_FeatureClipper_AlwaysReturnsValueInRange (Property-Test), Test_RoleAssigner_AlwaysReturnsValidRole (Property-Test), Test_AnalysisCache_RoundTrip_PreservesAllData

## Phase 7: Camelot-Kompatibilität und Score-Engine

- [ ] 7.1 `CamelotCompatibilityTable` implementieren: lädt die 24-Tonart-Kompatibilitätstabelle exakt aus `MixEngineConfig`, stellt `IsCompatible(string keyA, string keyB)` bereit
- [ ] 7.2 `ScoreCalculator` implementieren: `CalculateBaseCompat` (gewichtete Summe mit exakten Gewichten aus mixEngine.json), `CalculatePhaseBoost` (Phase-Boost-Werte: WARMUP+12, HYPNOSIS+14, LIFT+10, PEAK+8, RESET+11), `CalculateFinalScore` (Formel: base_compat*0.6 + phase_boost*0.2 + risk_penalty*0.2), `Classify` (Decision-Matrix: ≥85 PERFECT, 70-84 GOOD, 55-69 RISKY, <55 AVOID)
- [ ] 7.3 `PairRuleEngine` implementieren: alle 26 PAIR_01–PAIR_26 aus mixEngine.json; verbotene Paare (PAIR_04, PAIR_09, PAIR_20) liefern `Forbidden` mit Bridge-Kandidaten; erlaubte Paare liefern `Allowed` mit PreferredTransitionType, DurationBeats, MaxBpmDelta, KeyRule
- [ ] 7.4 `GlobalRuleEngine` implementieren: alle 25 GLOBAL_01–GLOBAL_25; Critical-Regeln als harte Constraints (GLOBAL_01, 03, 21, 23, 24); High/Medium als Penalties
- [ ] 7.5 `HardOverrideEngine` implementieren: alle 6 Kategorien (hook_conflict_high, bass_clash_high, double_break_risk_high, bridge_required, key_incompatible_melodic_overlap, low_confidence_analysis); wird VOR allen Score-Berechnungen ausgeführt
- [ ] 7.6 `RiskModel` implementieren: alle 7 Komponenten (hook_conflict, bass_clash, double_break_risk, brightness_jump, key_conflict, phrase_misalignment, low_analysis_confidence); Penalties exakt: RISK_01 -18, RISK_02 -20, RISK_03 -16, RISK_04 -8, RISK_05 -14, RISK_06 -12, RISK_07 -6
- [ ] 7.7 `DiversityTracker` implementieren: max. 3 gleiche Rollen in Folge, Penalty -15 danach; `preferred_break_roles` aus mixEngine.json
- [ ] 7.8 `PhaseAssigner` implementieren: weist Tracks den 5 Set-Phasen zu; erzwingt Mindestdauern (WARMUP 15min, HYPNOSIS 25min, LIFT 20min, PEAK 15min, RESET 10min) und Energie-Ziele
- [ ] 7.9 Unit-Tests Score-Engine (alle als Property-Tests mit FsCheck):
  - Test_ScoreFormula_MatchesExactFormula (Property-Test)
  - Test_DecisionMatrix_AllFourCategories
  - Test_CamelotCompatibility_IsSymmetric (Property-Test)
  - Test_CamelotCompatibility_SelfCompatible (Property-Test)
  - Test_AllPairRules_PAIR01_to_PAIR26 (26 Einzeltests)
  - Test_PAIR04_Foundation_MelodicOpen_IsForbidden
  - Test_PAIR09_HypnoticDriver_PsyEdge_IsForbidden
  - Test_PAIR20_MelodicOpen_Foundation_IsForbidden
  - Test_HardOverride_HookConflictHigh_ForcesLoopBlendOnly
  - Test_HardOverride_BassClashHigh_ForcesBassNeverOverlap
  - Test_HardOverride_DoubleBreakRiskHigh_ForcesBreakRebuildOnly
  - Test_HardOverride_BridgeRequired_InsertsBridgeTrack
  - Test_HardOverride_KeyIncompatibleMelodicOverlap_ForbidsHarmonicBlend
  - Test_HardOverride_LowConfidenceAnalysis_ReducesOverlap
  - Test_RiskModel_AllSevenPenalties (7 Einzeltests)
  - Test_DiversityTracker_ThreeSameRoles_NoPenalty
  - Test_DiversityTracker_FourSameRoles_AppliesMinus15 (Property-Test)

## Phase 8: Mixplaner

- [ ] 8.1 `BeamSearchPlanner` implementieren: BeamWidth=12, max. 12 Kandidaten pro Track-Position; Scoring via ScoreCalculator + PairRuleEngine + GlobalRuleEngine + HardOverrideEngine + RiskModel + DiversityTracker; Phasen-Zuweisung via PhaseAssigner; Bridge-Track-Insertion bei AVOID-Paaren; respektiert Locks (PositionLocked, IsStartTrack, IsEndTrack, gesperrte Übergänge)
- [ ] 8.2 `ConstraintRepairPlanner` implementieren: Fallback bei Beam-Search-Fehlschlag; greedy mit lokaler Reparatur; Bridge-Track-Insertion
- [ ] 8.3 `MixPlannerService` implementieren: orchestriert Planer, läuft als Hintergrundjob via `IJobService`, meldet Fortschritt „Berechne Mixplan – Schritt X/Y", unterstützt `CancellationToken`, speichert Ergebnis in SQLite
- [ ] 8.4 Alternative Mixvorschläge: generiert mindestens 3 alternative Pläne wenn Locks die Qualität einschränken
- [ ] 8.5 Unit-Tests Planer: Test_BeamSearch_ProducesValidPlan, Test_ConstraintRepair_FallbackWorks, Test_Locks_AreRespected, Test_BridgeTracks_InsertedForForbiddenPairs, Test_PhaseMinDurations_Enforced, Test_EmptyLibrary_ReturnsError, Test_LessThan3Tracks_ReturnsWarning

## Phase 9: Dynamic Transition Engine

- [ ] 9.1 `TransitionWindowDetector` implementieren: erkennt Outro-Kandidaten (outro_start, last_low_hook_phrase, post-break groove pocket, loopable_percussion_section) und Intro-Kandidaten (intro_start, first_full_kick_phrase, pre-hook phrase, low-risk tool section); erzwingt Phrasen-Alignment auf 8/16/32-Beat-Grenzen
- [ ] 9.2 `TransitionTypeSelector` implementieren: wendet DUR_01–DUR_05 aus mixEngine.json an; wählt bevorzugten Übergangstyp und Dauer; prüft `allowed_roles`, `bars_range` (exakt aus mixEngine.json: LONG_EQ_BLEND 32-64, INTRO_OVER_OUTRO 16-48, PERCUSSION_LOOP_BLEND 16-32, BREAK_REBUILD 8-16, BREAK_EXIT_BLEND 8-16, HARMONIC_BLEND 16-32, FILTER_SWAP 8-24, ECHO_OUT_SWITCH 4-8, TOOL_BRIDGE 16-32, SEAMLESS_TOOL 16-32, LOOP_BLEND_ONLY 8-16)
- [ ] 9.3 `StartPointCalculator` implementieren: wendet START_01–START_05 an; START_04 (brightness_jump>25 → HP-Filter), START_05 (low key_confidence → perkussiver Einstieg)
- [ ] 9.4 `BassSwapCalculator` implementieren: phrase_aligned_dynamic; erlaubte Swap-Punkte: 16, 8, 4 Beats vor Übergangsende; niemals >4 Beats Bassline-Overlap
- [ ] 9.5 `EqCurveCalculator` implementieren: Low sigmoid_or_piecewise, Mid linear_or_piecewise, High brightness_compensated
- [ ] 9.6 `FilterCurveCalculator` implementieren: HP auf eingehenden Track bei brightness_jump>25, LP auf ausgehenden Track bei melodic_conflict
- [ ] 9.7 `LoudnessCurveCalculator` implementieren: max. 2,0 LUFS Delta zwischen Tracks
- [ ] 9.8 `DynamicTransitionEngine` implementieren: orchestriert alle 7 Berechnungsschritte in exakter Reihenfolge aus mixEngine.json (validate_hard_constraints → detect_transition_windows → score_candidate_windows → select_transition_type → compute_exact_start_end → compute_automation_curves → apply_safety_overrides); befüllt alle 15 TransitionPlan-Schema-Felder
- [ ] 9.9 Unit-Tests Transition Engine: Test_AllTransitionPlanFields_AreFilled (Property-Test), Test_LufsDelta_NeverExceeds2 (Property-Test), Test_BassSwap_NeverExceeds4Beats (Property-Test), Test_PhraseAlignment_SnapsTo8_16_32, Test_BrightnessJump_TriggersHpFilter, Test_AllTransitionTypes_RespectBarsRange

## Phase 10: Rendering-Pipeline und MP3-Export

- [ ] 10.1 `PhaseVocoderTimeStretcher` implementieren: Phase-Vocoder-Algorithmus (NAudio SoundTouch-Wrapper oder äquivalent); artefaktfrei bis 6% BPM-Delta; chunk-basiert (max. 10 Sek.); CancellationToken-fähig
- [ ] 10.2 `TransitionRenderer` implementieren: wendet EQ-Automation (sample-genau, sigmoid/piecewise/brightness_compensated), Filter-Automation (HP/LP sample-genau), Bass-Swap (sample-genau am berechneten Beat-Offset), Lautstärke-Automation (max. 2,0 LUFS) an; Fehlerbehandlung: bei Fehler harter Schnitt als Fallback, Rendering fortsetzen
- [ ] 10.3 `MixBuffer` implementieren: chunk-basierter PCM-Puffer (max. 10 Sek. pro Chunk); schreibt in temporäre Datei
- [ ] 10.4 `FfmpegMp3Encoder` implementieren: ruft ffmpeg auf mit `ffmpeg -f s16le -ar 44100 -ac 2 -i pipe:0 -codec:a libmp3lame -b:a 320k -f mp3 [outputPath]`; liest PCM via stdin-Pipe; parst stderr für Fortschritt (time= Pattern → „MP3-Export X %"); behandelt Exit-Code ≠ 0 als Exception; löscht unvollständige Datei bei Abbruch/Fehler
- [ ] 10.5 `RenderingPipeline` implementieren: orchestriert alle 9 Rendering-Schritte (ValidateMixPlan → AudioDecoder → Resampler → TimeStretcher → TransitionRenderer → MixBuffer → PcmAccumulator → FfmpegMp3Encoder → ExportHistoryWriter); läuft als Hintergrundjob; meldet Fortschritt „Rendere Übergang X/Y – [Track A] → [Track B]"; unterstützt CancellationToken
- [ ] 10.6 `ExportService` implementieren: zeigt Datei-Speichern-Dialog, startet RenderingPipeline, schreibt ExportHistory in SQLite
- [ ] 10.7 Unit-Tests Rendering: Test_TimeStretching_PreservesLength (Property-Test), Test_BassSwap_NeverExceeds4Beats, Test_Mp3Export_Is320kbpsCbr, Test_CancellationToken_DeletesIncompleteFile, Test_RenderingError_FallsBackToHardCut
- [ ] 10.8 Integrationstest: Test_FullPipeline_ImportAnalyzePlanRenderExport_ProducesValidMp3

## Phase 11: GUI – WPF MVVM Black Theme

- [ ] 11.1 `BlackTheme.xaml` ResourceDictionary anlegen: Background #0A0A0A/#111111/#1A1A1A, Foreground #FFFFFF/#CCCCCC/#888888, Borders #2A2A2A/#3A3A3A, Statusindikatoren #CC3333/#888800/#336633; Schriftart Segoe UI 12px; keine bunten Akzentfarben
- [ ] 11.2 `MainWindow.xaml` anlegen: Dreispalten-Layout (GridSplitter), StatusBar am unteren Rand; Black Theme anwenden
- [ ] 11.3 `StatusBarViewModel` implementieren: CurrentJobText, ProgressPercent, IsRunning, CancelCommand, ErrorText, HasError; bindet an `IJobService`
- [ ] 11.4 `LibraryViewModel` implementieren: `ObservableCollection<TrackViewModel>`, Sortierung nach allen Spalten, Suchfilter (Dateiname/BPM/Tonart/Rolle), SelectedTrack; Commands: AnalyzeSelected, RemoveSelected, OverrideRole, ShowInExplorer
- [ ] 11.5 `LibraryView.xaml` anlegen: scrollbare ListView mit Spalten (Dateiname, Dauer, BPM, Tonart, Energie, Rolle, Analysezustand), Kontextmenü, Drag-&-Drop-Zone für Import; vollständig verdrahtet
- [ ] 11.6 `ImportViewModel` implementieren: DragDrop-Handler für Dateien und Ordner, ImportCommand, Fortschrittsanzeige
- [ ] 11.7 `SetlistViewModel` implementieren: `ObservableCollection<MixPlanEntryViewModel>`, Drag-&-Drop-Reordering, Lock-Visualisierung (Schloss-Symbol), Übergangsanzeige zwischen Tracks
- [ ] 11.8 `SetlistView.xaml` anlegen: ListView mit Drag-&-Drop-Reordering, Übergangs-Badges, Lock-Icons, Kontextmenü (Position Lock, Start/End Lock, Übergang sperren); vollständig verdrahtet
- [ ] 11.9 `PlanningViewModel` implementieren: PlanCommand, CancelPlanCommand, AlternativePlansCommand (min. 3 Alternativen), Fortschrittsanzeige
- [ ] 11.10 `TrackInspectorViewModel` implementieren: zeigt alle 22 Features, Rolle (Auto/Manuell/Lock-Status), WarnFlags, Konfidenz-Werte; RoleOverrideCommand, RoleLockCommand
- [ ] 11.11 `TransitionInspectorViewModel` implementieren: zeigt alle 15 TransitionPlan-Felder, HardOverridesApplied-Liste, RiskFlags mit Penalty-Werten, FinalScore, DecisionCategory
- [ ] 11.12 `WaveformViewModel` implementieren: lädt PeakData aus Cache, rendert als WriteableBitmap; Beatgrid-Overlay (Downbeats/Phrasengrenzen farblich unterschieden); Segment-Markierungen (Intro/Outro/Break/Build/Drop/Hook); Übergangs-Overlay (Überlappungsbereich, Bass-Swap-Punkt, EQ-Kurven); Zoom und Scroll; Peak-Daten-Berechnung im Hintergrund falls nicht gecacht
- [ ] 11.13 `InspectorView.xaml` anlegen: TabControl mit Track-Inspector, Transition-Inspector, Waveform-Ansicht; vollständig verdrahtet
- [ ] 11.14 `JobProgressViewModel` implementieren: IsRunning, ProgressText, ProgressPercent, CanCancel, CancelCommand; bindet an `IJobService` via `IProgress<JobProgress>`
- [ ] 11.15 Alle Fortschrittsanzeigen verdrahten: Import „Importiere Datei X/Y – [Dateiname]", Analyse „Analysiere Track X/Y – [Feature-Name]", Planung „Berechne Mixplan – Schritt X/Y", Rendering „Rendere Übergang X/Y – [Track A] → [Track B]", Export „MP3-Export X %"
- [ ] 11.16 Einstellungsdialog implementieren: Standard-Exportverzeichnis, maximale Analyse-Parallelität (1-4), FFmpeg-Pfad, Sprache; speichert in SQLite via SettingsRepository
- [ ] 11.17 Projektverwaltungs-Dialog implementieren: Neu/Laden/Speichern/Löschen; zeigt Liste mit Name/Erstellungsdatum/Änderungsdatum; behandelt fehlende Track-Dateien
- [ ] 11.18 DI-Host in `App.xaml.cs` konfigurieren: alle Services, Repositories und ViewModels registrieren; MixEngineConfigLoader beim Start aufrufen; FFmpeg-Verfügbarkeit prüfen; DatabaseMigrationService beim Start ausführen
- [ ] 11.19 GlobalExceptionHandler in `App.xaml.cs` einrichten: DispatcherUnhandledException + TaskScheduler.UnobservedTaskException; Logging via Serilog

## Phase 12: Logging und Fehlerbehandlung

- [ ] 12.1 Serilog konfigurieren: File-Sink nach `%APPDATA%\AutoDjMix\Logs\autodj_{date}.log`; Rotation 10 MB, max. 5 Dateien; Format `{Timestamp:yyyy-MM-dd HH:mm:ss.fff} [{Level}] [{SourceContext}] {Message}{NewLine}{Exception}`
- [ ] 12.2 `FFmpegNotFoundHandler` implementieren: beim Programmstart FFmpeg-Pfad prüfen; bei Fehlen M4A-Dekodierung und MP3-Export deaktivieren; Warnung in StatusBar anzeigen
- [ ] 12.3 `NAudioFallbackHandler` implementieren: bei NAudio-Codec-Fehler automatisch FFmpeg-Decoder verwenden; Warnung protokollieren
- [ ] 12.4 Fehlerbehandlung in allen Hintergrundjobs: Exceptions abfangen, protokollieren, via `IProgress<JobProgress>` an UI melden; Programm nicht beenden
- [ ] 12.5 SQLite-Fehlerbehandlung: bei Schreibfehler protokollieren, Fehlermeldung anzeigen, In-Memory-Betrieb fortsetzen; bei beschädigter DB beim Start: Backup anlegen, neue DB erstellen

## Phase 13: Vollständige Testsuite

- [ ] 13.1 Domain-Tests vervollständigen (AutoDjMix.Tests.Domain): alle 26 PairRule-Tests, alle 6 HardOverride-Tests, alle 7 RiskPenalty-Tests, ScoreFormula-Tests, CamelotCompatibility-Tests (alle 24 Tonarten), DiversityRule-Tests, FeatureClipper-Property-Tests
- [ ] 13.2 Application-Tests vervollständigen (AutoDjMix.Tests.Application): ImportService, AnalysisService, RoleAssignmentService, MixPlannerService, TransitionEngineService, RenderingService, ExportService
- [ ] 13.3 Integrationstests vervollständigen (AutoDjMix.Tests.Integration): FullPipeline (Import→Analyse→Plan→Render→MP3), DatabaseMigration, FfmpegIntegration, AnalysisCache-Persistenz
- [ ] 13.4 End-to-End-Tests vervollständigen (AutoDjMix.Tests.EndToEnd): E2E_FullSet_WithRealAudioFiles, E2E_CancellationToken_StopsAllJobs, E2E_Mp3Export_Is320kbpsCbr, E2E_MissingFiles_HandledGracefully, E2E_CorruptedAudio_FallsBackToFfmpeg, E2E_MissingFfmpegBinaries_DisablesFeatures, E2E_Analysis_10MinTrack_CompletesWithin120Seconds
- [ ] 13.5 Fehlerfälle testen: nicht lesbare Dateien, beschädigte Audiodaten, fehlende FFmpeg-Binaries, SQLite-Fehler, CancellationToken-Abbruch in allen Phasen
- [ ] 13.6 Performance-Test: Analyse eines 10-Minuten-Tracks auf Zielrechner in max. 120 Sekunden

## Phase 14: NSIS-Installer

- [ ] 14.1 `AutoDjMixSetup.nsi` anlegen: Sections Core (EXE + DLLs), FFmpeg (ffmpeg.exe → `%INSTDIR%\ffmpeg\`), Shortcuts (Desktop + Startmenü)
- [ ] 14.2 Uninstaller implementieren: entfernt `%INSTDIR%` und Verknüpfungen; behält `%APPDATA%\AutoDjMix\` (Benutzerdaten); optionale Löschung mit Bestätigung
- [ ] 14.3 Upgrade-Logik implementieren: prüft Registry-Key `HKLM\Software\Microsoft\Windows\CurrentVersion\Uninstall\AutoDjMix`; stille Deinstallation alter Version vor Installation; Benutzerdaten bleiben erhalten
- [ ] 14.4 Rollback bei Fehler implementieren: alle kopierten Dateien löschen; Fehlermeldung anzeigen
- [ ] 14.5 .NET 8 Desktop Runtime x64 Check implementieren: prüft Registry; zeigt Download-Link wenn nicht vorhanden
- [ ] 14.6 Registry-Einträge anlegen: DisplayName, DisplayVersion, Publisher, InstallLocation, UninstallString
- [ ] 14.7 Installer manuell testen: saubere Installation, Upgrade von Vorgängerversion, Deinstallation mit/ohne Benutzerdaten-Löschung
