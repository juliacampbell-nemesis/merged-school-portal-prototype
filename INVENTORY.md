# Merged TimelyCare + Alongside K-12 School Portal — Feature Inventory

> Scope-locking artifact for the wireframe prototype at
> `/Users/julia.campbell/Development/k12-monorepo/.claude/worktrees/upbeat-bohr/index.html`.
> Sources: TC school-portal codebase under `apps/school-portal/src/`, Alongside FigJam
> board `wbrtUBzGHNtJkrrE1mpRqS`, Alongside roles/permissions spreadsheet (prior),
> and screenshots already pulled. Anything I couldn't anchor in real evidence is
> tagged `[needs verification]` or `[inference]`.

## 1. Executive summary

The merged portal spans roughly **76 distinct staff-facing features** across
two source products: ~31 from the TimelyCare K-12 School Portal (roster,
visit lifecycle, invites, district/school admin, insights, insurance, settings)
and ~45 from Alongside (5 inbox alert types, dashboards, screeners, skills,
resources, reports, settings, Student Demo, 7 Self-Care surfaces). **Functional
overlap is low — about 15%** — limited to authentication, roster, role/permission
management, school directory, and reports (where Alongside's are richer). The
two products do fundamentally different things: TC delivers clinical visits;
Alongside delivers in-app screeners, AI chat, skills, and educator self-care.

The **seven bridge features** that justify the merger (and not just SSO) are:
(1) escalate Alongside alert → TC visit, (2) wellness-aware student row in
roster, (3) unified inbox combining alerts + visit notifications, (4) student
profile fusing Alongside trajectory + TC visit history, (5) single Clever
roster feeding both halves, (6) merged reports/dashboard pinning across
modalities, (7) staff-resource library bridging both toolkits. These are
where design effort produces the most value.

## 2. TimelyCare K-12 School Portal features

Grouped by area. Roles: DA = District Admin · DS = District Staff ·
SA = School Admin · SF = School Facilitator (School Staff) · TA = Timely Admin.
Permissions are from `packages/types/src/index.ts`.

### 2.1 Roster management

| Feature | File location | Description | PHI? | Roles | Key data fields |
|---|---|---|---|---|---|
| Student list (paginated, filterable) | `apps/school-portal/src/app/students/page.tsx` + `components/school/StudentList.tsx` | Search by name/ID/guardian, filter by grade/status, paginated table | No (basic info) | DA/DS/SA/SF (`student_management`) | firstName, lastName, grade, studentIdNumber, status |
| Add student (single) | `components/school/StudentForm.tsx` | Modal form, validation, optional Hybrid insurance fields | Yes (DOB, guardian) | DA/DS/SA (`student_management`; Hybrid schools need `view_edit_insurance_status`) | All Student fields + guardianInfo |
| Bulk student upload (CSV) | `components/students/BulkUploadModal.tsx`, `BulkUploadButton.tsx` | CSV import with row-by-row error reporting | Yes | DA/SA | StudentCSVRow |
| Edit student | `app/students/[id]/page.tsx` | Inline edit per section (student/guardian) | Yes | DA/SA | Same as add |
| Student detail page | `app/students/[id]/page.tsx` | Identity, guardian, insurance, visit history, invite history | Yes | DA/DS/SA/SF (`view_student_details`) | All Student fields |
| Deactivated students section | `components/school/DeactivatedStudentsSection.tsx` | Collapsible list, deactivatedOn/By | Yes | DA/SA | `deactivatedAt`, `deactivatedBy` |
| Archived students section | `components/school/ArchivedStudentsSection.tsx` | Roster reconciliation result of removal from upstream Clever sync | Yes | DA/SA | `archivedAt` |
| Insurance status badge / radio | `components/students/InsuranceStatusBadge.tsx`, `InsurancePermissionGatedButton.tsx` | Read-only badge or editable radio gated by `view_edit_insurance_status` | **Yes (PHI)** | DA/SA + explicit PHI grant | `insuranceStatus` (INSURED/UNINSURED/UNKNOWN), `insuranceCarrier` |
| View student clinical records | gate: `view_student_clinical_records` permission | Surfaces document section on student detail | **Yes (PHI)** | DA/SA, TA by default; SF requires explicit grant | `StudentDocumentsSection` |
| Student documents (care plan etc.) | `components/students/StudentDocumentsSection.tsx`, `CarePlanDocument.tsx`, `DynamicDocumentViewer.tsx` | Render documents for student | **Yes (PHI)** | gated as above | Document blobs |

### 2.2 Visit lifecycle

| Feature | File location | Description | PHI? | Roles | Key data fields |
|---|---|---|---|---|---|
| Schedule visit | `components/school/VisitScheduler.tsx` | Pick provider, date/time, duration, location (at_school/at_home/on_campus/day_student_off_campus) | Yes | `schedule_data` (DA/SA/TA) | scheduledDate, scheduledTime, duration, visitLocation, providerId |
| Today's visits (dashboard banner) | `components/dashboard/UpcomingVisitBanner.tsx`, used in `app/dashboard/page.tsx` | Next joinable visit with Join Call button | Yes | view: `schedule_data`/`visit_data`; join: `visit_facilitation` | Visit |
| Visits page (list, filters) | `app/dashboard/page.tsx` (renders unified list) | Filterable by status/date/type/location | Yes | `schedule_data`/`visit_data` | Visit + filters |
| Visit history (per student) | `components/students/VisitHistoryTable.tsx` | All past/upcoming for one student | **Yes (PHI)** | `visit_history` | Visit |
| Visit facilitation (join meeting) | dashboard banner + visit list | Join Call link uses `meetingLink` | Yes | `visit_facilitation` | `meetingLink` |
| Visit dashboard (district pacing) | `app/visit-dashboard/page.tsx` + `components/SchoolVisitDashboard.tsx`, `VisitBreakdownCard.tsx`, `PacingIndicator.tsx`, `PHIToggle.tsx` | District-wide table of allocated/scheduled/completed visits per school, pacing indicators | Yes | `visit_data` | VisitStats per school |
| PHI toggle on visit dashboard | `app/visit-dashboard/components/PHIToggle.tsx` | Reveal/hide PHI-rich columns | **Yes** | viewer-controlled at view time | UI state only |
| Visit pool view (per-school + per-student usage) | `app/visit-pool/page.tsx` + `SchoolPoolStatus.tsx`, `StudentUsageList.tsx` | Pool size, used, remaining, per-student cap | Yes | `visit_data` | poolSize, usedVisits, perStudentCap |

### 2.3 Invites / Referrals (TC's pre-visit flow)

| Feature | File location | Description | PHI? | Roles |
|---|---|---|---|---|
| Create invite (single, from student) | `components/school/AddReferralModal.tsx` + `ReferralForm.tsx` | Pick student, urgency, reason, notes | Yes | `referral_distribution` (PHI permission; explicit) |
| Invite distribution (send to guardian) | `lib/invite-reasons.ts`, `ReasonForInviteSection.tsx` | Reason taxonomy + delivery | Yes | `referral_distribution` |
| Invite management (view/track) | `components/students/ReferralHistoryTable.tsx`, `components/referrals/ReferralDetailModal.tsx`, `ReferralEditForm.tsx` | View status (PENDING/CLAIMED/etc.), edit pre-claim | Yes | `referral_management` |
| Invite history (per student) | `components/students/ReferralHistoryTable.tsx` + `InvitationHistoryTable.tsx` | Status timeline on student profile | Yes | `referral_management` |
| Referrals listing page | `app/referrals/page.tsx` + `app/referrals/[id]/page.tsx` | Full list of referrals across schools | Yes | `referral_management` |
| (Legacy) Guardian invitations page | `app/invitations/page.tsx` — **`@deprecated`** in code | Marked for deletion (verified Apr 28 2026) | — | — |

### 2.4 User & school admin

| Feature | File location | Description | PHI? | Roles |
|---|---|---|---|---|
| Users list (district-wide) | `app/settings/users/page.tsx` + `components/users/UserRow.tsx`, `MobileUserCard.tsx` | Table of all staff users with role, schools, status, last login | No | `school_user_management` (own school) or `district_user_management` (all) |
| Add user (invite) | `app/settings/users/add/page.tsx` → `add/review/page.tsx` | Two-step: details + permission review | No | `district_user_management` (for district-level roles) / `school_user_management` |
| Edit user / permissions | `app/settings/users/[id]/page.tsx` | Edit role, schools, permissions; activate/deactivate; resend invite | No | as above |
| School assignment modal | `components/users/SchoolAssignmentModal.tsx` | Multi-school assignment for school-level roles | No | `school_user_management` |
| User status badge | `components/users/UserStatusBadge.tsx` | ACTIVE / INACTIVE / INVITED | No | viewer |
| User actions dropdown | `components/users/UserActionsDropdown.tsx` | Edit, deactivate/activate, resend invite, school assignment | No | as above |
| Schools list (district view) | `app/settings/schools/page.tsx` | District admin: all schools, with visit/invite allocation | No | `school_referral_allocation` to manage |
| School detail / contact edit | `app/settings/schools/[id]/page.tsx` | Edit contact info, see stats, manage allocation | No | DA / SA for own school |
| Add school | `app/settings/schools/page.tsx` | Modal: name, state, primaryAdminName, contact, initialReferrals | No | DA |

### 2.5 Insights / Reports

| Feature | File location | Description | PHI? | Roles |
|---|---|---|---|---|
| Insights tab / dashboard | `app/insights/page.tsx` + `@k12/shared` `InsightsDashboard` | Cards for totalReferrals/Invites, completedVisits, averageWaitTime, atHomePercent etc. + provider performance table | Yes | `insights_tab_visibility` |
| Insights school filter | `app/insights/page.tsx` `SchoolFilterActions` | DA/multi-school users can filter by school | No | as above |
| Insights export | "Export Summary" button | CSV/PDF export | Yes | as above |
| Reports page (school-level table) | `app/reports/page.tsx` | Filterable by date range, referral status, visit status, provider | Yes | `view_reports` |
| Insurance-aware insights card | `app/insights/page.tsx` `insuranceCardVisible` | Hides insurance metrics if school config DISABLED | Yes | `view_edit_insurance_status` or `insurance_status_update_notifications` |

### 2.6 Settings / Misc

| Feature | File location | Description | PHI? | Roles |
|---|---|---|---|---|
| MFA setup (security) | `app/settings/security/mfa/setup/page.tsx`, `components/mfa/*` | TOTP/SMS setup, recovery codes | No | all users |
| Profile page | `app/profile/` (referenced from `app/dashboard`) | Edit own profile | No | all users |
| Notifications | `components/notifications/NotificationBell.tsx`, `NotificationDropdown.tsx` | In-app notification bell + dropdown | No | all users |
| Access Model banner | `components/school/AccessModelBanner.tsx` | Surfaces which model (POOL / INVITATION / REFERRAL) the school is on | No | all users |
| Feature flags | `lib/feature-flags/` | Runtime config | No | TA / engineering |
| Support / Timely contact info | `components/support/TimelyCareContactInfo.tsx` | Phone/email for support | No | all users |

### 2.7 Out of scope (TC)

| Feature | Why excluded |
|---|---|
| TC Internal Admin Portal (`apps/timely-dash`) | For TC ops staff, not school staff |
| Parent Portal (`apps/parent-portal`) | Guardian-facing, not in merger scope |
| Student Portal (`apps/student-portal`) | Replaced by Alongside student app |
| `/invitations` legacy page | Marked `@deprecated` in code (Apr 2026), pending deletion |

## 3. Alongside School Portal features

**Sources:** FigJam board `wbrtUBzGHNtJkrrE1mpRqS` (nodes 240:2180 Student Demo,
241:2203 Self-Care, 242:2206 user types), Alongside roles spreadsheet you
previously shared, and screenshots already loaded into context. I have **no
access to their codebase**, so every row below cites Figma/spreadsheet
evidence or is flagged otherwise. Roles in Alongside's matrix:
District Admin (DA), Principal (P), Asst Principal (AP), School Admin
(SchA), Counselor (C), Social Worker (SW), School Psychologist (SP),
Teacher (T), Other (O).

### 3.1 Inbox — 5 alert types

Confirmed from prior spreadsheet and the prototype's alerts table (which
mirrors Alongside's risk labels).

| Alert type | Evidence | Description | Trigger | Role gating |
|---|---|---|---|---|
| Crisis Email | Spreadsheet row "Crisis Email" | Email-only alert to subscribed staff for highest-risk events | Auto-detected high-risk language (e.g. suicidal ideation, C-SSRS) | All clinical roles (C/SW/SP/SchA/P) — staff opt-in by subscription |
| Crisis SMS | Spreadsheet row "Crisis SMS" | SMS-only escalation channel | Same as Crisis Email | Same as above; per-user subscription |
| Standard Message | Spreadsheet row "Standard Message" | Non-crisis automated message routed to staff (less urgent triggers) | Lower-risk indicators in chat | Same opt-in pattern |
| Anonymous Message | Spreadsheet row "Anonymous Message" | Student-submitted anonymous report — no student identity attached | Student initiates via "Someone else needs help" form | Routed to all subscribed staff |
| Chat summary | Spreadsheet row "Chat summary" | Student-initiated summary of what they learned/practiced in chat | Student finishes a skill/chat and chooses to share | Subscribers only |

**Alert sub-types referenced in prototype** (verified against Alongside risk
labels): Suicidal ideation, Risk of harm to others, Mental health concern,
Abuse by an adult, Someone else needs help, Insufficient information.
**Risk levels:** High, Moderate, Unknown.

### 3.2 Dashboard / Overview

| Feature | Evidence | Description | Roles |
|---|---|---|---|
| Overview / Insights tabs | Spreadsheet row "Overview / Insights tabs" | Two-tab structure for the staff-facing dashboard | All staff |
| "Data refreshes every hour" indicator | Spreadsheet row | Stale-data callout in the UI chrome | All staff |
| Students who completed (screener) | Spreadsheet row | Count of students who have completed a given screener | C/SW/SP/SchA |
| Results of screener | Spreadsheet row | Aggregate result distribution per screener | C/SW/SP/SchA |
| Dashboard count cards | Spreadsheet row "Dashboard" | Headline KPIs across alerts, screeners, engagement | All staff |
| School picker dropdown | Spreadsheet row "School picker dropdown" | Multi-school staff (e.g. district admin) switch schools | DA + multi-school staff |
| School name displayed at top | Spreadsheet row | Persistent "current school" affordance | All staff |
| Download SY25-26 Roster link | Spreadsheet row | Pre-populated roster download | School-admin level |

### 3.3 Students

| Feature | Evidence | Description | Roles |
|---|---|---|---|
| Student roster | Inferred from "Download Roster" + Overview tabs | Likely a sortable list with engagement signal | All staff |
| Student detail / engagement view | Spreadsheet "Chat summary", "Results of screener" per student | Per-student page with screener results, chat summaries, assigned skills | C/SW/SP/SchA |
| Per-student screener results | Spreadsheet "Results of screener" | View answers + scoring | C/SW/SP |
| Per-student chat summaries (history) | Spreadsheet "Chat summary" | Historical list of summaries student shared | C/SW/SP |
| Per-student assigned skills | Spreadsheet "Assign Skills" | View what skills a student has been assigned | C/SW/SP/T |
| Skill-assignment workflow | Spreadsheet "Assign Skills" | Choose skill, assign to student or cohort | C/SW/SP/T |

### 3.4 Screeners (library + delivery)

| Feature | Evidence | Description | Roles |
|---|---|---|---|
| Screener library | Spreadsheet "Screeners" | District-plan-driven list (e.g. PHQ-9, GAD-7, C-SSRS, PSC-17, SDQ) | All staff |
| Screener completion tracking | Spreadsheet "Students who completed" | Per-screener completion %, follow-up needed | C/SW/SP/SchA |
| Screener auto-assignment | Inferred from "automated based on district plan" note in TC prototype | Likely admin-configured cadence | [needs verification] |
| Rollout & Resources section | Spreadsheet "Rollout & Resources" | District-rollout planning materials | DA/SchA |

### 3.5 Skills library (library + assignment)

| Feature | Evidence | Description | Roles |
|---|---|---|---|
| Skills library browse | Spreadsheet "Assign Skills" + prototype mirror | Categorized skills (Coping, Cognitive, Mindfulness, Social, Sleep, Crisis) | All staff |
| Skill detail / preview | Inferred from "Assign" buttons | Read description, time estimate, completion rate, rating | All staff |
| Assign skill to student | Spreadsheet "Assign Skills" | Pick student(s), assign | C/SW/SP/T |
| Skill engagement aggregate | Inferred from prototype dashboard widget | Aggregate assigned/completed/abandoned | C/SW/SP/SchA |

### 3.6 Resources library

| Feature | Evidence | Description | Roles |
|---|---|---|---|
| Staff resources / Rollout & Resources | Spreadsheet "Rollout & Resources" | Onboarding, how-to guides, compliance docs, protocols | All staff |

### 3.7 Reports

The prototype's reports section (`#reports*`) heavily mirrors Alongside's
known reports surface — text labels like "Active accounts", "In-app hours of
support", "Activities completed", "Educator self-care", "Problems found",
"Goals created for skills" are pulled directly from Alongside's reporting UI.
[needs verification — anchored in observed screenshots, not source code.]

| Report | Evidence | Description |
|---|---|---|
| App usage (student + educator) | Prototype text matches Alongside reports screenshots | Active accounts, in-app hours, activities completed, helpfulness |
| Wellness (alerts trends) | Prototype `#reports-wellness` | Severe issues, safety plans created, alerts over time, alert type breakdown |
| Chat & topics | Prototype `#reports-chat` | "Problems found" topic distribution, "Goals created for skills" |
| Student mood & habits | Prototype `#reports-mood` | Aggregate mood/habit tracking analytics |
| Top content | Prototype `#reports-content` | Most-watched videos / most-used skills |
| Engagement mix | Prototype `#reports-engagement` | Mix of student activities |
| Student feedback | Prototype `#reports-feedback` | Quote-style feedback |
| Schools comparison | Prototype `#reports-schools` | Per-school KPI table |

### 3.8 Settings

| Feature | Evidence | Description |
|---|---|---|
| Settings | Spreadsheet "Settings" row | School/district configuration page |
| Log out | Spreadsheet "Log out" row | Standard sign-out |
| User management | Implied by roles spreadsheet | Add/edit staff, set role |
| Alert subscriptions | Spreadsheet "Crisis Email" / "Crisis SMS" being per-user | Each user opts into channels |
| District plan / Rollout config | Spreadsheet "Rollout & Resources" | District-level configuration |
| Integrations (Clever / SIS) | Inferred from "Download Roster" and SY25-26 references | Likely SIS sync configuration |

### 3.9 Student Demo (sandbox preview)

Confirmed from FigJam node `240:2180` (11 student-demo screenshots) and
spreadsheet "Student Demo" row.

| Surface | Evidence (Figjam screenshot) | Description |
|---|---|---|
| Demo Home | screenshot at x=293 | Mascot greeting, chat input, topic pills, "Recommended skills", "This week's habits" |
| Demo Habits | screenshot at x=1056 + 1772 | Monthly calendar with daily habit emoji + check-in modal (sleep / exercise / eating) |
| Demo Chat | screenshot at x=2480 | "Nova" chatbot, privacy disclaimer, topic pills |
| Demo Goals | screenshot at x=3478 | Empty state — "Create your first goal" |
| Demo Videos | screenshot at x=4477 | Grid: Get Motivated, Celebrities Tell All, Boost Your Mood, Calm and Focus |
| Demo Journal | screenshot at x=5440 + 6442 | Editor + Discover prompts side rail |
| Demo More | screenshot at x=7369 + 8064 + 9024 | Profile / Notifications / Settings / Privacy / Help / Get help now (24/7 crisis) |

Staff control: read-only preview, "Preview as: Grade X" dropdown, "Nothing
saves, no alerts fire" affordance.

### 3.10 Self-Care (educator's private space)

Confirmed from FigJam node `241:2203` (7 surfaces, per prior mapping):
**Home, Mood, Habits, Chat, Goals, Videos, More.** Visually mint sidebar +
purple CTAs (vs. orange CTAs in the student app). This is an educator-private
wellbeing space — staff log in to a personal/private mode separate from
Student Support.

| Surface | Description | Notes |
|---|---|---|
| Home | Mascot greeting, chat-with-Kiwi input, topic pills, trending videos for educators, This Week's Emotions snapshot, Recommended skills aside, Create-a-goal CTA | |
| Mood | Calendar + Insights tabs; mood-entry palette (16 emotions) | |
| Habits | Calendar of habit check-ins, daily check-in modal (sleep/exercise/eating) | Mirrors student habits |
| Chat | Private Kiwi chatbot | Privacy-first; no staff-back-channel |
| Goals | Goal-setting and tracking | |
| Videos | Educator-targeted content (e.g. "Box breathing for educators", "Compassion fatigue") | Distinct library from student |
| More | Profile, Notifications, Settings, Privacy, Help, Get help now | Crisis CTA still present |

Role gating: **all individual staff with a portal login** (DA/P/AP/SchA/C/SW/SP/T/O) — Self-Care is personal, not role-restricted.

## 4. Bridges — features only the merged product can offer

The merger's value, not just shared login. Each is a real workflow the
combined data model enables.

| Bridge | What it does | Alongside contributes | TC contributes |
|---|---|---|---|
| **Escalate Alongside alert → TC visit** | From an Inbox alert detail panel, staff click "Escalate to visit" → preselects student + reason → opens TC scheduler | Alert payload (type, risk, chat context, C-SSRS responses) | Visit scheduler, provider availability, allocation gate |
| **Wellness-aware roster row** | Student list shows a Wellness column (Stable / Monitoring / Urgent) AND TC visit cadence side by side | Wellness signal derived from screener + alert history | Visit count, last care date |
| **Unified inbox (alerts + visit notifications)** | One Inbox tab combines Alongside alerts + TC visit reminders/no-shows | Alert stream (Crisis Email/SMS/Standard/Anonymous/Chat summary) | Visit lifecycle events |
| **Student profile fusion** | Single profile page shows Alongside trajectory (screening history, chat summaries, alerts, assigned skills) alongside TC visit history + insurance + clinical docs (PHI-gated) | Screener trajectory, chat summaries, alert history, skill assignment | Visit history, care plan docs, insurance |
| **Single Clever roster sync feeding both** | One roster ingest → drives Alongside student accounts AND TC referral/invite eligibility | Student accounts for chat/skills/screeners | Referral/invite distribution, visit scheduling |
| **Pinned-widget dashboard across both modalities** | Reports → My Dashboard lets staff pin widgets from any source (TC visits delivered + Alongside wellness alerts + skill engagement) on one canvas | Wellness/usage/chat widgets | Care delivery, visit utilization widgets |
| **Merged staff resources library** | Library → Staff Resources cross-references Alongside skill best-practices AND TC facilitation guides | Skill / screener / chat docs | Facilitation, telehealth tech, allocation primer |
| **Cross-permission gate for clinical follow-up** | Severe screener result (Alongside) auto-suggests creating a TC referral; gated by `referral_distribution` PHI permission | Screener result + severity | Referral creation, PHI-gated send |
| **Crisis card on student profile** | Severe-risk Alongside event surfaces a single TC-routed CTA ("Escalate to Visit") on the student detail page | Risk detection | Same-day on-site visit pathway |

## 5. Status map (the actionable artifact)

`Status` values: `done` (visually complete) · `partial — <gap>` (some content, key gaps) · `stub — heading only` · `not built` · `out of scope`.
`Priority` values: P0 = must-have for handoff · P1 = should-have · P2 = nice-to-have · `out of scope`.

| Feature | Source | Status in prototype | Priority | Notes |
|---|---|---|---|---|
| **Inbox** | | | | |
| Alerts list (table) | Alongside | done | P0 | 11 rows, 5 alert types, risk badges match Alongside taxonomy |
| Alert detail panel (status, basic info, C-SSRS, chat context) | Alongside | done | P0 | Right rail, "Escalate to visit" CTA wired |
| Chat summaries list | Alongside | done | P0 | 7 rows, preview text, timestamps |
| Chat summary detail panel + "AI-suggested email" composer | Alongside | done | P0 | Email subject/body editable; "Email student" CTA |
| Alert email/SMS routing config (subscription preferences) | Alongside | not built | P1 | Settings → Alert subscriptions per user |
| **Home (Dashboard)** | | | | |
| KPI cards (alerts, chat summaries, today's visits, utilization) | Bridge | done | P0 | 4 cards combining both sources |
| Recent alerts widget | Alongside | done | P0 | 3 rows + View all |
| Recent chat summaries widget | Alongside | done | P0 | 3 rows + View all |
| Today's facilitated visits widget | TC | done | P0 | 4 rows with status pills |
| Screener follow-ups widget | Alongside | done | P0 | Aggregate by severity |
| Skill engagement widget | Alongside | done | P0 | Assigned/completed/abandoned |
| Synced-from-Clever timestamp | Bridge | done | P1 | Top-right of home page |
| **Students** | | | | |
| Student list with wellness column | Bridge | done | P0 | Adult ineligibility for invites is correctly disabled |
| Filters (grade, status, wellness) | Bridge | done | P0 | Wellness filter is bridge-specific |
| Bulk invite sheet (Create invite flow) | TC | done | P0 | Review state → sending state → done state |
| Student detail — identity card | TC | done | P0 | Status pill, guardian inline |
| Student detail — active crisis card | Bridge | done | P0 | "Escalate to Visit" CTA wired |
| Student detail — alerts section (Alongside) | Bridge | done | P0 | 4-row table, links to Inbox |
| Student detail — visit history (TC) | TC | done | P0 | 5 rows including no-show |
| Student detail — chat summaries (Alongside) | Bridge | done | P0 | 3 rows |
| Student detail — screening trajectory (Alongside) | Alongside | done | P0 | Bar chart + 4 readings |
| Student detail — assigned skills section | Alongside | not built | P1 | Listed in dropdown menu only |
| Student detail — care plan / documents (PHI) | TC | not built | P1 | "View care plan" in dropdown, no panel |
| Student detail — insurance status badge + edit | TC | not built | P2 | Hybrid-model-only; not needed for v1 unless we target Hybrid |
| Schedule visit modal | TC | not built | P0 | "Schedule visit" button is dead |
| Bulk invite progress / confirmation flow | TC | done | P0 | Already in prototype |
| Single student "Send Invite" from row | TC | not built | P1 | Only bulk path works |
| Add student modal (new student) | TC | not built | P2 | Clever-sync covers most; manual add is edge case |
| Edit student modal | TC | not built | P2 | Same reasoning |
| Deactivated students collapsible | TC | not built | P2 | Edge case |
| **Schools (district view)** | | | | |
| Schools list (district level) | TC | done | P0 | 5 rows: name, level, students, staff, crises, visits used |
| Add school modal | TC | not built | P1 | DA-only action |
| School detail — header (principal, counselor lead, contact) | TC | done | P0 | |
| School detail — Overview tab (KPIs) | TC | done | P0 | 4 metric cards |
| School detail — Staff list | TC | done | P0 | 4 rows with role badges |
| School detail — Visit utilization card | TC | done | P0 | |
| School detail — Staff tab | TC | stub — heading only | P1 | Tab present but no panel content |
| School detail — Visit allocation tab | TC | stub — heading only | P1 | Editable allocation form needed |
| School detail — Crisis history tab | Bridge | stub — heading only | P1 | Alongside alerts scoped to this school |
| School detail — Contact tab | TC | stub — heading only | P2 | Edit contact info form needed |
| Add staff modal (from school detail) | TC | not built | P1 | Inline alt to Settings → Users |
| **Library** | | | | |
| Skills tab (catalog grid) | Alongside | done | P0 | 6 skill cards w/ assigned/done/rating |
| Skills assign modal | Alongside | not built | P0 | "Assign" button is dead — primary skill action |
| Skill preview modal | Alongside | not built | P1 | "Preview" button is dead |
| Screeners tab (table) | Alongside | done | P0 | 5 screeners (PHQ-9, GAD-7, PSC-17, SDQ, C-SSRS) |
| Screener detail / results drill-down | Alongside | not built | P1 | Row click does nothing |
| Send screener (manual) | Alongside | not built | P1 | Library is "read-only" per the inline note — may not be needed |
| Staff resources tab (cards) | Bridge | done | P0 | 6 category cards w/ link lists |
| **Reports** | | | | |
| Reports launcher / categories | Bridge | done | P0 | Care, Wellness, Engagement, District, Activity |
| My Dashboard (pinned widgets) | Bridge | done | P0 | 3 pinned sections + empty-state hint |
| Care delivery report | TC | done | P0 | Charts + visit mix |
| Wellness report | Alongside | done | P0 | KPIs + alert types + alerts over time |
| App usage report (student + educator) | Alongside | done | P0 | Student + educator self-care KPIs |
| Chat & topics report | Alongside | done | P0 | Problems found + Goals created |
| Mood & habits report | Alongside | partial — heading + filters, light content | P1 | Real Alongside data viz richer than current stubs |
| Top content report | Alongside | partial — same | P1 | Same |
| Engagement mix report | Alongside | partial — same | P1 | Same |
| Student feedback report | Alongside | partial — same | P2 | Quote-style content |
| Schools comparison report | Bridge | partial — same | P1 | Per-school KPI table |
| Recent downloads / scheduled reports | Alongside | partial — same | P2 | Activity section in launcher |
| Report download (CSV/PDF) modal | Bridge | not built | P1 | "Download" button visible everywhere but inert |
| Schedule a report (email cadence) | Alongside | not built | P2 | "Scheduled emails" button visible but inert |
| Filter widget global / per-section | Bridge | partial — buttons present, no panel | P1 | Critical for live data |
| **Student Demo** | | | | |
| Demo Home | Alongside | done | P0 | Mascots + chat + recommended skills + habits rail |
| Demo Habits | Alongside | done | P0 | Calendar + check-in modal |
| Demo Chat (Nova) | Alongside | done | P0 | Header, privacy banner, topic pills |
| Demo Goals (empty state) | Alongside | done | P0 | |
| Demo Videos | Alongside | done | P0 | 4 categories |
| Demo Journal | Alongside | done | P0 | Editor + Discover rail |
| Demo More | Alongside | done | P0 | 6 cards including Get-help-now |
| Demo "Preview as: Grade X" selector | Alongside | done | P0 | Static but visible |
| Demo "Nothing saves" affordance | Bridge | done | P0 | Slim orange ribbon at top |
| **Self-Care (My Wellness)** | | | | |
| SC Home (mascot, chat, topics, trending vids, week emotions) | Alongside | done | P0 | Faithful 1:1 |
| SC Mood (calendar + entry) | Alongside | done | P0 | |
| SC Habits | Alongside | partial — calendar only | P1 | Insights tab not built |
| SC Chat | Alongside | partial — header only [needs verification] | P1 | May need conversation history |
| SC Goals | Alongside | partial — needs verification of empty state | P1 | |
| SC Videos | Alongside | partial — limited content | P1 | |
| SC More | Alongside | partial — limited content | P1 | |
| Switch to Student Support button | Bridge | done | P1 | Cross-mode affordance |
| **Settings** | | | | |
| My profile section | Bridge | done | P0 | Form with role + permission chips |
| Look & feel (theme/density/landing) | Bridge | done | P1 | Personal pref |
| Users & permissions table | TC | done | P0 | 9 rows, multi-school, INVITED state |
| Add/invite user modal | TC | not built | P0 | "Invite user" CTA inert |
| User row actions (edit/deactivate/resend) | TC | not built | P0 | Kebab menu inert |
| Settings → Notifications panel | Bridge | stub — sidebar link only | P1 | Per-user alert subscriptions live here |
| Settings → Connected accounts | Bridge | stub — sidebar link only | P2 | OAuth/Clever |
| Settings → Privacy & data | Bridge | stub — sidebar link only | P2 | |
| Settings → District profile | TC | stub — sidebar link only | P1 | DA-only |
| Settings → Roles | TC | stub — sidebar link only | P1 | Role-permission editor |
| Settings → Clever / SSO integration | Bridge | stub — sidebar link only | P1 | Sync schedule, last sync |
| Settings → Audit log | TC | stub — sidebar link only | P2 | |
| Settings → Billing | TC | stub — sidebar link only | out of scope | DA only, not core workflow |
| Settings → Help & support | Bridge | stub — sidebar link only | P2 | |
| **Cross-cutting** | | | | |
| Hash routing | Bridge | done | P0 | All `#route` links work |
| Sidebar persistent nav | Bridge | done | P0 | |
| Account menu dropdown | Bridge | done | P0 | |
| MFA setup flow | TC | not built | out of scope | Already exists in real app |
| Notification bell + dropdown | TC | not built | P2 | Currently no bell in header |
| Access model banner | TC | not built | P2 | Edge case for live app |
| **Excluded** | | | | |
| TC Internal Admin Portal | TC | out of scope | — | Different app (`apps/timely-dash`) |
| Parent Portal | TC | out of scope | — | Different app |
| Student Portal (TC version) | TC | out of scope | — | Replaced by Alongside student app |
| TC `/invitations` deprecated page | TC | out of scope | — | Marked `@deprecated` in code |
| Hybrid insurance Carrier collection | TC | out of scope | — | Hybrid-model districts only |
| Boarding-school visit locations (`on_campus`, `day_student_off_campus`) | TC | out of scope | — | Boarding-school-only |

## 6. Open questions for Julia

These are the unknowns that prevent finalizing scope. Resolving each unlocks
specific build decisions.

1. **Who is the canonical primary user of v1?** The prototype uses "Julia C., Counselor Lead, Lincoln High" — a single school facilitator. Do we also need a district-admin variant (different home, schools list as default, district-wide reports) before handoff, or is one-persona-only acceptable for v1?

2. **Which Alongside roles map to which TC permission set?** The TC types are
   {DISTRICT_ADMIN, DISTRICT_STAFF, SCHOOL_ADMIN, SCHOOL_FACILITATOR, TIMELY_ADMIN}.
   Alongside's are {District Admin, Principal, Asst Principal, School Admin,
   Counselor, Social Worker, School Psychologist, Teacher, Other}. The roles
   matrix doc you have probably resolves this — but it's not encoded anywhere
   in the prototype's role guards yet. **Decision needed:** keep TC's 5 roles
   as the canonical set and treat Alongside roles as job-title labels? Or
   expand to 9 roles?

3. **Where does PHI live in the bridge features?** Specifically: when an Alongside chat-summary email composer is opened (currently in the Inbox detail panel), is that email content PHI? Today the panel is wide open. **Decision needed:** does the bridge inherit TC's `phi`/`view_student_clinical_records` gate?

4. **Insurance / Hybrid model — keep, defer, or drop for v1?** TC has substantial Hybrid-Commercial-Model logic (`useInsurancePermissionGate`, `view_edit_insurance_status` permission, carrier collection). Prototype currently has no insurance UI. If we don't target Hybrid districts in v1, all those features stay `out of scope`.

5. **Schedule-visit flow — full Healthie scheduler or stub?** TC's real scheduler (`VisitScheduler.tsx`) talks to providers via Healthie. For wireframe, do we need a 4-step modal (provider → date/time → duration → location) or is a static "Schedule visit" placeholder sufficient?

6. **Skills "Assign" flow — single-student or cohort?** Alongside spreadsheet says "Assign Skills" — but the data model could be either. **Decision needed:** assign-to-cohort UX, assign-to-single-student UX, or both?

7. **Reports: filter granularity and date pickers — real or stub?** Every report has a "Filter" button and "Year to date" pill that are inert. For a "fully functioning" wireframe, do we need the filter panel populated and date picker working?

8. **Student Demo "Preview as: Grade X" — does grade actually swap content?** Right now it's a static select. If yes, we need 3 sets of mock skills/videos. If no, it's a static affordance.

9. **Self-Care app — separate route, or a "mode switch" inside the same shell?** Currently it's both — `#my-wellness` is its own page with a tinted-mint sidebar. Is that the final intended pattern, or should it be a tab inside Settings → My Wellness?

10. **Alert subscription model — per-user opt-in (Alongside), per-role default (TC), or a hybrid?** The current Settings → Notifications stub doesn't pick a model. **Decision needed:** does a Crisis SMS auto-go to all Counselors, or do Counselors opt in via Settings?

11. **Teacher role — alerts visibility?** Alongside's matrix probably restricts which alert types Teachers see. The current Inbox shows all alerts to all viewers. **Decision needed:** what (if anything) does a Teacher persona see in Inbox vs. Counselor?

12. **Bridge "Add TC clinician to Alongside care plan"** — was raised in your prompt as an example. I couldn't find concrete evidence this exists in either product. **Decision needed:** is this a future feature we should design for, or speculative?

13. **Single-source-of-truth for student wellness signal** (Stable / Monitoring / Urgent). Currently in prototype as a column on the roster. Where does this derive from? Alongside screener severity? Most recent alert risk? A combined heuristic? Definition needed before the wellness filter can be honest.

14. **Bulk Invite sheet — does it survive the merger?** In TC today, "Invite" means guardian-invite for visit consent. In the merged world with Clever roster sync, are invites still needed (and for what — guardian consent? Alongside student-account activation?), or does the workflow collapse?
