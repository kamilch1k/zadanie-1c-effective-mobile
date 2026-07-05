# FINISH GUIDE — do this top-to-bottom when you're back at the PC

English UI labels (your Designer is English). Object **names** stay Russian —
the code binds to them. Whole thing is ~15 minutes. Nothing here needs me; it's
all keyboard-on-Designer work I can't do remotely.

Legend: `Tree` = the Configuration tree on the left. Rhythm for sub-items is
always: **expand the node → right-click the sub-branch → Add**.

---

## Phase 1 — Metadata (~8 min)

### 1.0 Fix the enum you already made
Open **Enumerations → СтатусыДокументов → Values**. Verify the 3rd value is
spelled exactly **`Акцептован`** (not "Акцентован"). If wrong, double-click it →
fix Name. The four values must be: `Создан`, `Внутренний`, `Акцептован`, `Заблокирован`.

### 1.1 Catalog `Договоры`
`Tree` → right-click **Catalogs → Add**.
- **Data** page → **Name length = 100**.
- Expand node → right-click **Attributes → Add**:
  - `Активен` — Type = **Boolean**
  - `НачалоДействия` — Type = **Date**; in Properties set **Date composition = Date**

### 1.2 Document `Ведомость`
`Tree` → right-click **Documents → Add**.
- **Numbering** page → Number type = **String**, Number length = **11**, **uncheck Autonumbering**.
- Expand node → right-click **Tabular sections → Add** → name `Договоры`.
  - Expand that tabular section → right-click **Attributes → Add**:
    - `Договор` — Type = **CatalogRef.Договоры**

### 1.3 Information register `ИсторияСтатусовДокументов`
`Tree` → right-click **Information registers → Add**.
- In Properties: **Writing mode = Independent**, **Periodicity = Nonperiodical**.
- Expand node → right-click **Dimensions → Add** (three of them):
  - `ДатаЗаписи` — Type = **Date**; **Date composition = Date and time**
  - `Документ` — Type dialog → tick **Composite data type** → check **DocumentRef.Ведомость**
  - `Статус` — Type = **EnumRef.СтатусыДокументов**
- No resources.

### 1.4 Subsystem (so objects are reachable in Enterprise mode)
`Tree` → **Common** → right-click **Subsystems → Add** → name `Данные`.
- **Content** tab → tick `Договоры`, `Ведомость`, `ИсторияСтатусовДокументов`.

### 1.5 Apply
**Configuration menu → Update Database Configuration (F7)** → accept changes.

---

## Phase 2 — External data processor (~5 min)

**File → New → External data processor.** Name = `ОбработкаВедомостей`.

### 2.1 Main form
Right-click **Forms → Add → Managed form**, name `Форма`, mark as main.

**Attributes** tab:
- `ТаблицаВедомостей` — **Value table**, add columns:
  | Column | Type |
  |---|---|
  | `Ведомость` | DocumentRef.Ведомость |
  | `Номер` | String (20) |
  | `ДатаВедомости` | Date (Date and time) |
  | `КоличествоДоговоров` | Number (10) |
  | `ПоследнийСтатус` | EnumRef.СтатусыДокументов |
- `ВариантСравнения` — **Number** (1); Choice list: `0 → «равна»`, `1 → «не равна»`
- `ВариантЭкстремума` — **Number** (1); Choice list: `0 → «минимальной»`, `1 → «максимальной»`

**Commands** tab: `ЗаполнитьНужное`, `УдалитьЛишнее`.

**Elements** (drag onto the form):
- Drag `ТаблицаВедомостей` → the table appears. On columns `Номер`, `ДатаВедомости`,
  `КоличествоДоговоров`, `ПоследнийСтатус` → uncheck **Visible** (only `Ведомость` shows).
- Drag `ВариантСравнения` → set element **View = Radio button field**. Same for `ВариантЭкстремума`.
- Drag both commands into the form command bar.

**Module:** paste all of `ОбработкаВедомостей.Форма.МодульФормы.bsl`.
(Conditional formatting is set in code — don't touch the designer for it.)

### 2.2 Message form
Right-click **Forms → Add → Managed form**, name `ФормаСообщения`.
- **Parameters** tab: `Текст` — **String**.
- **Attributes**: `ТекстСообщения` — **String**, Unlimited length.
- Drag `ТекстСообщения` onto the form → Multiline = Yes, ReadOnly = Yes, stretch.
- Form **Title** = `Результат обработки`.
- Paste `ОбработкаВедомостей.ФормаСообщения.МодульФормы.bsl`.

---

## Phase 3 — Seed test data (~1 min, one click)

1. On the **main form**, add a temporary command `ЗагрузитьТест` + a button for it.
2. Paste all of `ЗаполнитьТестовыеДанные.bsl` into the main form module, and make
   the `ЗагрузитьТест` command call `ЗаполнитьТестовыеДанныеНаСервере()`.
3. **File → Save** the processor as `ОбработкаВедомостей.epf`.
4. Run it (Enterprise mode → File → Open → the `.epf`) and click **ЗагрузитьТест once**.
   → 15 contracts, 8 documents, 17 register rows created.
5. **Remove** the `ЗагрузитьТест` command + the pasted filler code, save again.
   (It's scaffolding, not part of the deliverable.)

---

## Phase 4 — Test (~2 min)

Open the `.epf` in Enterprise mode.
1. Defaults «равна» + «минимальной» → **Заполнить нужное**. Expect, in order:
   `БТ02-000016, ЛР04-000256, БТ02-000032, ЛР04-001024, БТ02-000128`;
   rows `БТ02-000016` and `ЛР04-001024` light-green.
2. **Удалить лишнее** → popup: `Удалено документов: 4 (БТ02-000016, БТ02-000032, ЛР04-000256, ЛР04-001024).`
   Table keeps `БТ02-000128`.
3. Switch to «максимальной» → Заполнить нужное → only `БТ02-000016` remains (Example 2).

> If today's day-of-month equals a statement's day (sample days: 1,2,10,12,16,20),
> your personal rule (1) will drop that statement — that's correct, not a bug.
> To compare cleanly against the PDF, comment out the `ТекущийДень` block.

> If `ОткрытьФорму(ИмяФормыСообщения()...)` errors, hardcode the name:
> `"ВнешняяОбработка.ОбработкаВедомостей.Форма.ФормаСообщения"`.

---

## Phase 5 — Ship (~2 min)

1. Copy `ОбработкаВедомостей.epf` into the repo folder (`C:\cc\zadanie_1c`).
2. Optional but nice: Designer → right-click the processor → **Save to files** into
   the repo, so the code is readable on GitHub.
3. Commit + push:
   ```
   git add -A && git commit -m "Готовая обработка + выгрузка исходников" && git push
   ```
4. Submit the repo URL at https://effective-mobile.ru/forms/tz_junior/ (specialization = 1С).

Done. Everything above is the only hands-on-keyboard work left; all code is written and tested-in-logic.
