# Osysharp.ShiftScheduling

**A staff rota.** Who works when — for a café, a clinic, a shop floor, a venue, a warehouse, a volunteer team.

- **Shifts** with a number of places, a position ("Front desk", "Kitchen"), a break and a place — built as **drafts** and
  **published** in one go, so people hear about their week once.
- **Places** a manager assigns or a worker **claims** from the open shifts — never over-filled, one place a person, no
  overlaps — and every refusal a sentence: *You are on Closing then.* · *Bo is off on 3 Nov.* · *Espresso bar needs
  Espresso training: no record of the course.*
- **What a position requires** ("food-hygiene", "forklift"), answered by your app — or by
  [Osysharp.Qualifications](https://github.com/osysharp), which arrives with `Osysharp.ShiftScheduling.Qualifications` by itself.
- **Your own rules** (`IShiftRule`), found without wiring: *Trainees never work a shift alone.*
- **Time off** asked for and approved (the manager is told which shifts it touches), and the **hours each week someone is
  never free** — a claim is refused then; a manager is asked to confirm.
- **An hours cap** per week, as a warning a manager can go past knowingly.
- **Cover and swaps**: ask anybody, ask one colleague, or offer a swap for their shift — taken, then approved (or not);
  dropping a shift far enough ahead where you allow it.
- **Hours** per person and period, less breaks; **copy last week**; **cancel** a shift and keep who was on it.
- **A reminder before every shift**, durable: it waits while the shift is a draft, re-sets when it moves, and stops when
  the place is given up.

| A manager's week | A worker's shifts |
|---|---|
| ![the week: seven days of shifts, how full each is, drafts to publish, everybody's hours](docs/rota-week.png) | ![a worker's shifts, cover they could take and open shifts to claim](docs/rota-my-shifts.png) |

| Requests | Positions |
|---|---|
| ![time off and cover waiting for an answer](docs/rota-requests.png) | ![positions and what each requires](docs/rota-positions.png) |

| On a phone · in the dark |
|---|
| ![a worker's shifts on a phone](docs/rota-my-shifts-phone.png) ![the week in the dark theme](docs/rota-week-dark.png) |

## Use it

```osy
// app.osy
use Osysharp.Identity@0;
use Osysharp.ShiftScheduling@0;
```

```osy
using Osysharp.Identity;
using Osysharp.ShiftScheduling;

policy ManagesShifts => IsManager;    // builds and publishes the week, assigns, approves time off and cover
policy WorksShifts => IsStaff;      // sees the published rota, claims open places, asks for time off and cover

app.ShiftScheduling = new ShiftSchedulingSetup { TimeZone = "Europe/London", ReminderLead = TimeSpan.FromHours(12), MaxHoursPerWeek = 40m,
                           WorkersMayDrop = true, CoverNeedsApproval = true };

// The app's own rule — found, never wired.
public class NoDoubleCloses : IShiftRule {
  public string? Refusal(Shift shift, Principal worker) { … return null; }
}

// Telling people: the kit decides when, the app decides how.
[On] void Remind(ShiftReminderDue e) { /* send e.Assignment.Worker a message */ }
[On] void Published(ShiftsPublished e) { /* tell each of e.Workers their week is out */ }
```

From code: `AddShift`, `UpdateShift`, `PublishShifts(from, to)`, `CancelShift`, `CopyWeek`, `AssignShift(shift, worker)`,
`ClaimShift(shift)`, `WhyNotOnShift(shift, worker)`, `Unassign`, `DropShift`, `RequestCover`, `RequestSwap`,
`AcceptCover`, `DecideCover`, `RequestTimeOff`, `DecideTimeOff`, `MarkUnavailable`, `HoursBetween(from, to)`,
`ShiftsOf(worker)`, `OpenShiftsFor(worker)`. Events: `ShiftsPublished`, `ShiftAssigned`, `ShiftUnassigned`,
`ShiftChanged`, `ShiftCancelled`, `ShiftReminderDue`, `CoverRequested`, `CoverTaken`, `ShiftCovered`, `CoverDeclined`,
`TimeOffRequested`, `TimeOffDecided`. Give staff their roles with Osysharp.UserManagement.

## Pages

`/shifts` (the signed-in person), `/staff/rota` (the week), `/staff/rota/requests` and `/staff/rota/positions` are served
in the app's working shell, in its theme. Take one over by declaring a component of the same name, or place `MyShifts`,
`RotaWeek`, `RotaRequests`, `RotaPositions` or `ShiftCardView` on a page of your own.

## Personal data

Why somebody asked for time off or cover is classified personal content. In an app with Osysharp.Privacy,
Osysharp.Privacy.ShiftScheduling arrives by itself: a person's shifts, time off, unavailability and cover requests are in their
copy of their data and are erased when they ask.

## What informed it

- **Deputy** — open shifts the first eligible person claims; swaps that exchange a shift for a colleague's future one,
  only between people with the training it needs, with manager approval optional; unavailability and leave beside the
  schedule; changes reaching staff only when published.
- **When I Work** — dropping a shift versus swapping it versus releasing it; drops that wait for approval; a notice
  period before a drop.
- **RosterElf and Dolce** — a warning about the hours a change adds before a manager approves it.
- **Rotacloud, Planday and Sling** — a week built as a draft and published at once, copied forward, with hours per person
  for payroll.

## Source

[github.com/osysharp/shiftscheduling](https://github.com/osysharp/shiftscheduling). Its tests (`tests/`) are a café's: every
behaviour above, each seam shown, and the pages under `osy test --pixels`.

## Examples

Taken from this package's own tests, which compile and run on every release.

```osy
using Osysharp.Identity;
using Osysharp.UserAccounts;
using Osysharp.ShiftScheduling;

[Test(Cafe)]
[runas(Bo)]
void a_barista_claims_an_open_place_and_gets_the_lowest_free_one() {
  var first = ClaimShift(ShiftNamed("Opening"));
  Assert.True(first.IsAssigned);
  Assert.Equal("You are on Opening.", first.Message);
  Assert.Equal(1, first.Assignment!.Slot);
  runas(Ada) { Assert.Equal(2, ClaimShift(ShiftNamed("Opening")).Assignment!.Slot); }
  runas(Cy) {
    var full = ClaimShift(ShiftNamed("Opening"));
    Assert.Equal(ShiftResult.Refused, full.Result);
    Assert.Equal("This shift is full.", full.Message);
  }
  Assert.Equal(0, OpenPlaces(ShiftNamed("Opening")));
}
```

From [`tests/assigning.test.osy`](tests/assigning.test.osy).

```osy
using Osysharp.Identity;
using Osysharp.UserAccounts;
using Osysharp.ShiftScheduling;

[Test(Cafe)]
[runas(Ada)]
void cover_is_taken_by_a_colleague_and_the_lead_approves_it() {
  var mine = OnShift(Ada, "Opening");
  var request = RequestCover(mine, "Sick child");
  runas(Bo) {
    Assert.Equal(1, CoverOffersFor(Bo).Count);
    var taken = AcceptCover(request);
    Assert.Equal(CoverState.Taken, taken.State);
  }
  Assert.Equal(Ada.Id, ShiftAssignment.Single(a => a.Id == mine.Id).Worker.Id);
  runas(Dana) {
    Assert.Equal(1, CoverAwaitingApproval().Count);
    DecideCover(request, true);
    var now = ShiftAssignment.Single(a => a.Id == mine.Id);
    Assert.Equal(Bo.Id, now.Worker.Id);
    Assert.Equal(1, now.Slot);
  }
  Background.Settle();
  Assert.Equal(1, Heard.Count(h => h.Event == "covered" && h.Detail == "Opening:Bo Lindqvist"));
  Assert.Equal(1, HeardCount("covertaken"));
}
```

From [`tests/cover.test.osy`](tests/cover.test.osy).

```osy
using Osysharp.Identity;
using Osysharp.UserAccounts;
using Osysharp.ShiftScheduling;
using Osysharp.Workflow;

[Test(Cafe)]
[runas(Dana)]
void a_draft_week_is_invisible_to_baristas_until_it_is_published() {
  OnShift(Ada, "Weekend brunch");
  OnShift(Bo, "Weekend brunch");
  runas(Ada) {
    Assert.Equal(0, Shift.Count(s => s.Title == "Weekend brunch"));
    Assert.Equal(0, ShiftsOf(Ada).Count);
  }
  Assert.Equal(1, PublishShifts(At(0, 0), At(7, 0)));
  runas(Ada) { Assert.Equal(1, ShiftsOf(Ada).Count); }
  Background.Settle();
  Assert.Equal(1, Heard.Count(h => h.Event == "published" && h.Detail == "1:2"));
}
```

From [`tests/schedule.test.osy`](tests/schedule.test.osy).

## Learn more

- [The Osysharp.ShiftScheduling reference](https://osysharp.com/reference/ui/shift-scheduling-kit/)
- [Osysharp.ShiftScheduling in the package catalogue](https://osysharp.com/packages/#osysharp-shiftscheduling)
- [The Osy# reference](https://osysharp.com/reference/) — the language, the runtime and every kit
- [`llms-full.txt`](https://osysharp.com/llms-full.txt) — the whole reference and specification as one Markdown file, for a coding agent

## Licence

MIT — see [`LICENSE`](LICENSE).
