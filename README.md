# Yuval-Keren.github.io
<style>
* { box-sizing: border-box; margin: 0; padding: 0; }
body { font-family: var(--font-sans, sans-serif); }
.uml-root { width: 100%; padding: 12px 0; }
.layer-tabs { display: flex; gap: 6px; flex-wrap: wrap; margin-bottom: 16px; }
.layer-tab {
  padding: 6px 14px; border-radius: 20px; font-size: 12px; font-weight: 500;
  cursor: pointer; border: 1.5px solid transparent; transition: all .18s;
  color: var(--color-text-secondary);
  background: var(--color-background-secondary);
  border-color: var(--color-border-tertiary);
}
.layer-tab:hover { border-color: var(--color-border-primary); color: var(--color-text-primary); }
.layer-tab.active { color: #fff; border-color: transparent; }
.layer-tab[data-layer="gui"].active     { background: #0F6E56; }
.layer-tab[data-layer="app"].active     { background: #534AB7; }
.layer-tab[data-layer="services"].active{ background: #185FA5; }
.layer-tab[data-layer="domain"].active  { background: #993C1D; }
.layer-tab[data-layer="infra"].active   { background: #5F5E5A; }

.layer-desc {
  font-size: 12px; color: var(--color-text-secondary);
  margin-bottom: 14px; padding: 8px 12px;
  border-left: 3px solid var(--color-border-secondary);
  border-radius: 0 6px 6px 0;
  background: var(--color-background-secondary);
}
.diagram-area { width: 100%; overflow-x: auto; }
.diagram-area svg { display: block; }

.cls-card { cursor: pointer; }
.cls-card rect { transition: filter .15s; }
.cls-card:hover rect.cls-bg { filter: brightness(1.06); }
.cls-card text { pointer-events: none; }

.detail-panel {
  margin-top: 14px; padding: 14px 16px;
  border: 1px solid var(--color-border-tertiary);
  border-radius: 10px; background: var(--color-background-secondary);
  font-size: 13px; line-height: 1.6; display: none;
}
.detail-panel.show { display: block; }
.detail-title { font-size: 14px; font-weight: 500; color: var(--color-text-primary); margin-bottom: 6px; }
.detail-role { color: var(--color-text-secondary); font-size: 12px; margin-bottom: 8px; }
.detail-members { display: grid; grid-template-columns: 1fr 1fr; gap: 4px 16px; }
.detail-member { font-size: 12px; color: var(--color-text-secondary); font-family: var(--font-mono, monospace); }
.detail-member.field  { color: var(--color-text-info); }
.detail-member.method { color: var(--color-text-success); }
.legend { display: flex; gap: 14px; flex-wrap: wrap; margin-bottom: 12px; font-size: 11px; color: var(--color-text-secondary); }
.legend-dot { display: inline-block; width: 10px; height: 10px; border-radius: 3px; margin-right: 4px; vertical-align: middle; }
</style>

<div class="uml-root">
  <div class="layer-tabs" id="tabs"></div>
  <div class="legend" id="legend"></div>
  <div class="layer-desc" id="layerDesc"></div>
  <div class="diagram-area" id="diagramArea"></div>
  <div class="detail-panel" id="detailPanel">
    <div class="detail-title" id="dpTitle"></div>
    <div class="detail-role"  id="dpRole"></div>
    <div class="detail-members" id="dpMembers"></div>
  </div>
</div>

<script>
const LAYERS = {
  gui: {
    label: "GUI Layer",
    color: "#0F6E56", light: "#E1F5EE", stroke: "#1D9E75",
    desc: "PyQt6 widgets, screens, presenters, and UI components. The GUI never talks directly to domain objects — it reads ViewModels via presenters and fires controller commands.",
    classes: [
      { id:"App",                  role:"Main QMainWindow / entry point",    fields:["-_controller: AppController","-_router: ScreenRouter"], methods:["+__init__(controller)","+closeEvent(event)"] },
      { id:"ScreenRouter",         role:"Screen navigation manager",         fields:["-_stack: QStackedWidget","-_screens: Dict","-_history: list"], methods:["+register(name,screen)","+show(name)","+back()"] },
      { id:"Screen",               role:"Abstract base screen",              fields:[], methods:["+on_enter()","+on_leave()"] },
      { id:"InputScreen",          role:"File load + config UI",             fields:["-_presenter: InputScreenPresenter"], methods:["+on_enter()","+show_progress(n)","+show_done(n)"] },
      { id:"InputScreenPresenter", role:"Input screen logic coordinator",    fields:["-_view","-_controller","-_router","-_courses_loaded: bool","-_periods_loaded: bool"], methods:["+on_generate_clicked(ids)","+on_schedule_found(n)","+on_search_finished()","+on_open_program_dialog()"] },
      { id:"ActionBarWidget",      role:"File loader + run/cancel bar",      fields:[], methods:["+set_ready()","+set_running(n)","+set_cancelled()"] },
      { id:"ProgramSelectorDialog",role:"Program picker popup",              fields:["-_selected_ids: List[str]"], methods:["+selected_ids(): List[str]","+exec()"] },
      { id:"ProgramSelectorCardWidget", role:"Single program toggle card",   fields:["-_program_id: str","-_checked: bool"], methods:["+is_checked(): bool","+toggle()"] },
      { id:"OutputScreen",         role:"Schedule viewer screen",            fields:["-_presenter: OutputScreenPresenter"], methods:["+on_enter()","+show_schedule(vm)","+show_period(vm)"] },
      { id:"OutputScreenPresenter",role:"Output screen logic coordinator",   fields:["-_view","-_controller","-_router","-_current_index: int","-_total: int","-_periods: PeriodNavigator"], methods:["+on_next()","+on_prev()","+on_page_flip(dir)","+on_export_pdf()","+on_period_apply(vms)"] },
      { id:"SolutionBarWidget",    role:"Nav + export + paging toolbar",     fields:[], methods:["+set_count(n,total)","+set_page_info(page,total)"] },
      { id:"PeriodNavigator",      role:"Exam period navigation helper",     fields:["-_periods: List[PeriodEditViewModel]","-_current_index: int"], methods:["+next()","+prev()","+current(): PeriodEditViewModel","+set_periods(vms)"] },
      { id:"CalendarEditorWidget", role:"Period date exclusion editor",      fields:["-_view_models: List[PeriodEditViewModel]"], methods:["+toggle_date_exclusion(date_str)","+get_view_models(): List[PeriodEditViewModel]"] },
      { id:"CalendarWidget",       role:"Month grid display",                fields:[], methods:["+setup_month_grid(dates)","+display_assignments(items)","+set_date_excluded_style(date,on)"] },
      { id:"OutputCalendarWidget", role:"Read-only output calendar",         fields:[], methods:["+display_schedule(vm)"] },
      { id:"CourseListWidget",     role:"Scrollable course list panel",      fields:["-_courses: List"], methods:["+load_courses(courses)","+selected_ids(): List[str]"] },
      { id:"HeaderWidget",         role:"App header / branding bar",         fields:[], methods:["+set_title(text)"] },
      { id:"SchedulePdfExporter",  role:"PDF writer module",                 fields:[], methods:["+export_schedule_pdf(view,idx,parent)","-_build_schedule_html(view)","-_write_html_to_pdf(html,path)"] },
    ]
  },
  app: {
    label: "App Layer",
    color: "#534AB7", light: "#EEEDFE", stroke: "#7F77DD",
    desc: "The controller and facade coordinate all user actions. AppController receives GUI signals; ApplicationFacade orchestrates services. Neither contains business logic.",
    classes: [
      { id:"AppController",    role:"GUI command handler (QObject)",  fields:["-_facade: ApplicationFacade","-_worker: SchedulerWorker"], methods:["+load_file(path,type,mode)","+generate_schedules(program_ids)","+cancel_scheduling()","+update_exam_periods(vms)","+get_schedule_view(index)","+get_page_info()","+on_app_closing()"] },
      { id:"ApplicationFacade",role:"Service orchestrator",           fields:["-_state: AppState","-_importer: FileImportService","-_scheduler: SchedulingService","-_exporter: ScheduleExportService","-_mapper: ViewModelMapper"], methods:["+import_file(request): ImportResult","+generate(program_ids): SchedulerWorker","+update_periods(vms)","+get_schedule_vm(index)","+export(index,path)","+load_page(page)"] },
      { id:"AppState",         role:"Composite state root",           fields:["-_input_state: InputDataState","-_schedule_state: ScheduleResultState"], methods:["+get_input_state()","+get_schedule_state()"] },
      { id:"ImportRequest",    role:"Value object — import params",   fields:["+path: str","+file_type: str","+mode: ImportMode"], methods:[] },
      { id:"ImportResult",     role:"Value object — import outcome",  fields:["+success: bool","+loaded_count: int","+errors: List[str]"], methods:["+has_errors()"] },
      { id:"ImportBoundary",   role:"ImportMode enum + interface",    fields:["ImportMode.REPLACE","ImportMode.APPEND"], methods:[] },
      { id:"ScheduleDTO",      role:"Picklable schedule transfer obj",fields:["+assignments: List[AssignmentDTO]","+total_assignments: int"], methods:[] },
      { id:"AssignmentDTO",    role:"Picklable assignment transfer",  fields:["+course_id: str","+course_name: str","+date: str","+semester: str","+moed: str"], methods:[] },
      { id:"ScheduleDTOAdapter",role:"Adapter: DTO → Domain",        fields:["-assignments: List"], methods:["+to_exam_schedule(): ExamSchedule"] },
    ]
  },
  services: {
    label: "Services Layer",
    color: "#185FA5", light: "#E6F1FB", stroke: "#378ADD",
    desc: "Stateless application services — each owns a single responsibility. They transform data, orchestrate domain calls, and communicate via DTOs and ViewModels.",
    classes: [
      { id:"FileImportService",    role:"Parse + validate + merge files",    fields:["-_cache: InputCacheService","-_parser_factory: ParserFactory","-_validators: ValidatorPipeline","-_merger: InputDataMerger","-_state: InputDataState"], methods:["+load_file(path,type,mode): ImportResult"] },
      { id:"InputCacheService",    role:"Cache load / persist façade",       fields:["-_repository: IDataRepository","-_detector: FileChangeDetector"], methods:["+try_load(paths)","+persist(state,paths)"] },
      { id:"InputDataMerger",      role:"Merge loaded data into state",      fields:["-_state: InputDataState","-_handlers: Dict"], methods:["+merge(data,mode)"] },
      { id:"SchedulingService",    role:"Build slots + spawn workers",       fields:["-_repository: SQLiteScheduleRepository","-_slot_builder: SlotBuilder"], methods:["+build_slots(progs,courses,periods)","+generate_async(...): SchedulerWorker","+cancel()"] },
      { id:"ScheduleExportService",role:"Save / format schedules",           fields:["-_writer: OutputWriter"], methods:["+save(schedule_dto,path)","+format(schedule_dto): str"] },
      { id:"ViewModelMapper",      role:"Domain → ViewModel transform",      fields:[], methods:["+to_schedule_vm(dto,idx,total,progs)","+to_calendar_vm(dto)","+to_period_edit_vms(periods)","+to_program_vms(courses)"] },
      { id:"SchedulerWorker",      role:"QThread worker — drains queue",     fields:["-_queue: Queue","-_cancel_event: Event","-_processes: List[Process]","-_repository: SQLiteScheduleRepository"], methods:["+run()","+cancel()","-_handle_schedule_batch(payload)","-_shutdown()"] },
      { id:"InputDataState",       role:"Loaded input data holder",          fields:["-_courses: List[Course]","-_periods: List[ExamPeriod]","-_selected_program_ids: List[str]"], methods:["+replace_courses(courses)","+replace_periods(periods)","+apply_period_edits(vms)","+to_cache(): DataCache","+load_cache(cache)"] },
      { id:"ScheduleResultState",  role:"In-memory schedule results",        fields:["-_schedules: List[ScheduleDTO]","-_current_index: int"], methods:["+set_schedules(s)","+get_schedule(index)","+count()","+total_pages()"] },
      { id:"HybridScheduleResultState", role:"SQLite-backed paged results",  fields:["-_repository: SQLiteScheduleRepository","-_current_page: int"], methods:["+load_page(page)","+get_schedule(index)","+count()","+total_pages()"] },
    ]
  },
  domain: {
    label: "Domain Layer",
    color: "#993C1D", light: "#FAECE7", stroke: "#D85A30",
    desc: "Pure Python — no frameworks, no I/O. The scheduling algorithm, conflict checkers, slot builder, observer interfaces, and domain entities live here.",
    classes: [
      { id:"Scheduler",                role:"Backtracking search engine",      fields:["-_checkers: List[IConflictChecker]"], methods:["+generateSchedules(slots,observer,max)","-_backtrack(index,slots,...)"] },
      { id:"SlotBuilder",              role:"Course → candidate slot list",    fields:["-_period_map: dict","-_selected_set: Set[str]"], methods:["+build(courses): List[Slot]","-_score(slot)"] },
      { id:"Slot",                     role:"Course scheduling candidate",     fields:["+course","+semester","+moed","+candidateDates: List[date]"], methods:[] },
      { id:"IConflictChecker",         role:"«interface» Conflict check",      fields:[], methods:["+check(assignment,schedule): bool"] },
      { id:"ProgramYearConflictChecker",role:"Program/year conflict check",    fields:["-_conflict_graph"], methods:["+precompute_conflicts(courses,progs)","+check(...): bool"] },
      { id:"MoedOrderChecker",         role:"Moed A must precede Moed B",      fields:["-_moed_rank: dict"], methods:["+check(...): bool"] },
      { id:"IScheduleObserver",        role:"«interface» Observer contract",   fields:[], methods:["+on_schedule_found(schedule)","+on_progress(value)","+should_cancel()","+on_finished()","+on_error(msg)"] },
      { id:"CollectingScheduleObserver",role:"Collects all results in-memory", fields:["-_schedules: List"], methods:["+on_schedule_found(s)","+get_schedules()"] },
      { id:"StreamingScheduleObserver",role:"Streams results directly to disk",fields:["-_output_path","-_writer"], methods:["+on_schedule_found(s)"] },
      { id:"Course",                   role:"Domain entity",                   fields:["+courseId: str","+name: str","+instructor: str","+evaluation: EvalType","+programEntries: List[ProgramEntry]"], methods:["+hasExam(): bool"] },
      { id:"ProgramEntry",             role:"Value object — program link",     fields:["+programId: str","+year: int","+semester: Semester","+requirement: Requirement"], methods:[] },
      { id:"ExamPeriod",               role:"Domain entity",                   fields:["+semester","+moed","+startDate","+endDate","+excludedDates: Set[date]","+availableDates: List[date]"], methods:[] },
      { id:"ExamSchedule",             role:"Domain entity — assignment list", fields:["+assignments: List[ExamAssignment]"], methods:["+addAssignment(a)","+removeAssignment(a)","+getByDate(date)"] },
      { id:"ExamAssignment",           role:"Value object",                    fields:["+course","+date","+moed","+semester"], methods:[] },
    ]
  },
  infra: {
    label: "Infrastructure",
    color: "#5F5E5A", light: "#F1EFE8", stroke: "#888780",
    desc: "Everything touching disk, SQLite, multiprocessing IPC, file hashing, and parsing. No business logic — pure I/O adapters the service layer depends on via interfaces.",
    classes: [
      { id:"SQLiteScheduleRepository", role:"Schedule persistence (SQLite)",  fields:["-_db_path: str","-_total_count: int","-_lock: Lock"], methods:["+insert_batch(batch)","+insert_compressed_batch(data,count)","+get_window(offset,limit)","+count()","+clear()"] },
      { id:"QueueScheduleObserver",    role:"Observer → IPC queue adapter",   fields:["-_queue: Queue","-_cancel_event: Event","-_buffer: List[ScheduleDTO]","-_batch_size: int"], methods:["+on_schedule_found(s)","-_flush_buffer()","+should_cancel(): bool"] },
      { id:"SchedulerProcessRunner",   role:"Worker process entry point",      fields:["-_slots: List[Slot]","-_checkers: List[IConflictChecker]","-_queue: Queue","-_cancel_event: Event"], methods:["+run()","-_create_observer()","-_create_scheduler()"] },
      { id:"CachedInputLoader",        role:"Cache-or-parse loader helper",    fields:["-_repository: IDataRepository","-_detector: FileChangeDetector","-_course_parser","-_period_parser"], methods:["+load(paths): Tuple[List[Course],List[ExamPeriod]]"] },
      { id:"DataCache",                role:"«dataclass» Serializable cache",  fields:["+courses","+periods","+source_hashes","+schema_version"], methods:[] },
      { id:"DiskCacheRepository",      role:"Pickle-based cache on disk",      fields:["-_cache_path: Path"], methods:["+save(cache)","+load(): DataCache"] },
      { id:"IDataRepository",          role:"«interface» Cache repo",          fields:[], methods:["+save(cache)","+load(): DataCache"] },
      { id:"FileChangeDetector",       role:"Hash-based change detection",     fields:[], methods:["+compute_hash(path): str","+compute_hashes(paths): dict","+has_changed(paths,stored): bool"] },
      { id:"FileParser",               role:"Abstract file parser",            fields:[], methods:["+parse(file_path)","+validateSeparator(content,sep)"] },
      { id:"CoursesFileParser",        role:"Parses courses.txt",              fields:[], methods:["+parse(file_path): List[Course]"] },
      { id:"ExamPeriodsFileParser",    role:"Parses periods.txt",              fields:[], methods:["+parse(file_path): List[ExamPeriod]"] },
      { id:"ProgramsFileParser",       role:"Parses programs.txt",             fields:[], methods:["+parse(file_path): List[str]"] },
      { id:"ParserFactory",            role:"Creates correct parser by type",  fields:["-_REGISTRY: Dict"], methods:["+create(file_type): FileParser","+parse_files(mappings)","+register(type,cls)"] },
      { id:"ValidatorPipeline",        role:"Runs all validators in chain",    fields:["-_validators: List[IInputValidator]"], methods:["+add(validator)","+validate(data,fail_fast)"] },
      { id:"MaxProgramsValidator",     role:"Max 5 programs guard",            fields:["+MAX_PROGRAMS = 5"], methods:["+validate(...): bool"] },
      { id:"ProgramExistenceValidator",role:"Programs exist in master",        fields:["-_valid_ids: set"], methods:["+validate(...): bool"] },
      { id:"ValidationResult",         role:"«dataclass» Validation result",   fields:["+is_valid: bool","+errors: List[str]"], methods:["+add_error(msg)","+merge(other)"] },
      { id:"OutputWriter",             role:"Abstract schedule writer",        fields:[], methods:["+write(schedules,path)","+formatSchedule(schedule)"] },
      { id:"TextFileWriter",           role:"Writes schedules as .txt",        fields:[], methods:["+write(schedules,path)","+formatSchedule(schedule,caches)"] },
    ]
  }
};

const LEGEND_ITEMS = [
  { color: "#0F6E56", label: "GUI" },
  { color: "#534AB7", label: "Application" },
  { color: "#185FA5", label: "Services" },
  { color: "#993C1D", label: "Domain" },
  { color: "#5F5E5A", label: "Infrastructure" },
];

const legendEl = document.getElementById('legend');
LEGEND_ITEMS.forEach(l => {
  legendEl.innerHTML += `<span><span class="legend-dot" style="background:${l.color}"></span>${l.label}</span>`;
});

let activeLayer = 'gui';
const tabsEl = document.getElementById('tabs');
Object.entries(LAYERS).forEach(([key, layer]) => {
  const btn = document.createElement('button');
  btn.className = 'layer-tab' + (key === activeLayer ? ' active' : '');
  btn.dataset.layer = key;
  btn.textContent = layer.label;
  btn.onclick = () => setLayer(key);
  tabsEl.appendChild(btn);
});

function setLayer(key) {
  activeLayer = key;
  document.querySelectorAll('.layer-tab').forEach(t => t.classList.toggle('active', t.dataset.layer === key));
  document.getElementById('detailPanel').classList.remove('show');
  renderDiagram(key);
}

function renderDiagram(key) {
  const layer = LAYERS[key];
  document.getElementById('layerDesc').textContent = layer.desc;
  const classes = layer.classes;
  const COL = 3;
  const W = 210, GAP_X = 18, GAP_Y = 14, PAD_X = 16, PAD_Y = 20;
  const LINE_H = 13;
  const HEADER_H = 42;
  const DIVIDER_Y = 38;

  function cardHeight(cls) {
    const fc = Math.min(cls.fields.length, 3);
    const mc = Math.min(cls.methods.length, 4);
    const extra = (cls.fields.length > 3 ? 1 : 0) + (cls.methods.length > 4 ? 1 : 0);
    const total = fc + mc + extra;
    return HEADER_H + (total > 0 ? 8 + total * LINE_H + 6 : 10);
  }

  const rows = Math.ceil(classes.length / COL);
  const rowHeights = [];
  for (let r = 0; r < rows; r++) {
    let maxH = 0;
    for (let c = 0; c < COL; c++) {
      const idx = r * COL + c;
      if (idx < classes.length) maxH = Math.max(maxH, cardHeight(classes[idx]));
    }
    rowHeights.push(maxH);
  }

  const totalW = COL * W + (COL - 1) * GAP_X;
  const startX = Math.max(PAD_X, (680 - totalW) / 2);
  const svgH = PAD_Y + rowHeights.reduce((a, h) => a + h + GAP_Y, 0) - GAP_Y + PAD_Y + 4;
  const svgW = 680;

  let svg = `<svg width="100%" viewBox="0 0 ${svgW} ${svgH}" role="img">`;
  svg += `<title>${layer.label} UML classes</title>`;
  svg += `<desc>Interactive class diagram for the ${layer.label} — click any card to expand details</desc>`;
  svg += `<style>
    .cls-card text { font-family: var(--font-sans, sans-serif); }
    .cls-card .cls-bg { transition: opacity .12s; }
    .cls-card:hover .cls-bg { opacity: .82; }
    .cls-card { cursor: pointer; }
    .cls-divider { stroke: ${layer.stroke}; stroke-width: 0.5; opacity: .5; }
    .cls-title { font-size: 13px; font-weight: 500; fill: ${layer.color}; }
    .cls-role  { font-size: 10px; fill: ${layer.stroke}; }
    .cls-field { font-size: 10px; fill: #185FA5; }
    .cls-method{ font-size: 10px; fill: #3B6D11; }
    .cls-border{ fill: ${layer.light}; stroke: ${layer.stroke}; stroke-width: 0.8; }
    .cls-count { font-size: 9px; fill: ${layer.stroke}; opacity: .7; }
  </style>`;

  let rowY = PAD_Y;
  for (let r = 0; r < rows; r++) {
    const rh = rowHeights[r];
    for (let c = 0; c < COL; c++) {
      const idx = r * COL + c;
      if (idx >= classes.length) break;
      const cls = classes[idx];
      const x = startX + c * (W + GAP_X);
      const y = rowY;
      const H = rh;
      const fc = Math.min(cls.fields.length, 3);
      const mc = Math.min(cls.methods.length, 4);

      svg += `<g class="cls-card" onclick="showDetail('${key}','${cls.id}')">`;
      svg += `<rect class="cls-bg cls-border" x="${x}" y="${y}" width="${W}" height="${H}" rx="8"/>`;
      svg += `<line class="cls-divider" x1="${x+8}" y1="${y+DIVIDER_Y}" x2="${x+W-8}" y2="${y+DIVIDER_Y}"/>`;
      svg += `<text class="cls-title" x="${x+W/2}" y="${y+15}" text-anchor="middle" dominant-baseline="central">${cls.id}</text>`;
      const roleStr = cls.role.length > 34 ? cls.role.slice(0,33)+'…' : cls.role;
      svg += `<text class="cls-role" x="${x+W/2}" y="${y+29}" text-anchor="middle" dominant-baseline="central">${roleStr}</text>`;

      let lineY = y + HEADER_H + 4;
      for (let f = 0; f < fc; f++) {
        const lbl = cls.fields[f].replace(/[<>]/g,'');
        svg += `<text class="cls-field" x="${x+8}" y="${lineY}" dominant-baseline="central">${lbl.length>31?lbl.slice(0,30)+'…':lbl}</text>`;
        lineY += LINE_H;
      }
      if (cls.fields.length > 3) {
        svg += `<text class="cls-count" x="${x+8}" y="${lineY}" dominant-baseline="central">+${cls.fields.length-3} more fields…</text>`;
        lineY += LINE_H;
      }
      if (mc > 0) lineY += 2;
      for (let m = 0; m < mc; m++) {
        const lbl = cls.methods[m].replace(/[<>]/g,'');
        svg += `<text class="cls-method" x="${x+8}" y="${lineY}" dominant-baseline="central">${lbl.length>31?lbl.slice(0,30)+'…':lbl}</text>`;
        lineY += LINE_H;
      }
      if (cls.methods.length > 4) {
        svg += `<text class="cls-count" x="${x+8}" y="${lineY}" dominant-baseline="central">+${cls.methods.length-4} more methods…</text>`;
      }
      svg += `</g>`;
    }
    rowY += rh + GAP_Y;
  }
  svg += `</svg>`;
  document.getElementById('diagramArea').innerHTML = svg;
}

function showDetail(layerKey, classId) {
  const cls = LAYERS[layerKey].classes.find(c => c.id === classId);
  if (!cls) return;
  const layer = LAYERS[layerKey];
  document.getElementById('dpTitle').style.color = layer.color;
  document.getElementById('dpTitle').textContent = cls.id;
  document.getElementById('dpRole').textContent = cls.role;
  const el = document.getElementById('dpMembers');
  el.innerHTML = '';
  cls.fields.forEach(f => {
    const d = document.createElement('div');
    d.className = 'detail-member field';
    d.textContent = f;
    el.appendChild(d);
  });
  cls.methods.forEach(m => {
    const d = document.createElement('div');
    d.className = 'detail-member method';
    d.textContent = m;
    el.appendChild(d);
  });
  document.getElementById('detailPanel').classList.add('show');
}

renderDiagram('gui');
</script>
