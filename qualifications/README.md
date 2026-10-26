# Osysharp.ShiftScheduling.Qualifications

A rota position's requirements answered by the qualifications people hold. In an app using
[Osysharp.ShiftScheduling](https://github.com/osysharp) and [Osysharp.Qualifications](https://github.com/osysharp), a position
requiring `first-aid` refuses anybody whose first aid certificate does not hold **on the day of the shift**, in the
qualifications kit's own words: *First aid post needs a first aid certificate: First aid certificate expired on 5 July
2026.*

## Use it

Nothing to write. It arrives by itself when an app uses both kits (`osy lock` includes it and says so); to keep it out,
`without Osysharp.ShiftScheduling.Qualifications;` in `app.osy`. A requirement's key is a qualification kind's key. The check reads
as the caller, so let whoever builds the rota read qualifications:

```osy
policy ChecksQualifications => IsManager;
```

Another answer of the app's own (`IShiftRequirements`) can stand beside it: a requirement is met when either confirms it.

## Source

[github.com/osysharp/shiftscheduling/qualifications](https://github.com/osysharp/shiftscheduling/tree/main/qualifications); its tests are a festival medical
tent's.

## Examples

Taken from this package's own tests, which compile and run on every release.

```osy
using Osysharp.Identity;
using Osysharp.UserAccounts;
using Osysharp.ShiftScheduling;
using Osysharp.Qualifications;

[Test(Tent)]
[runas(Ada)]
void a_volunteer_whose_certificate_holds_on_the_day_takes_the_post() {
  Assert.True(ClaimShift(Shift.Single(s => s.Title == "Main stage post")).IsAssigned);
}
```

From [`tests/tent.test.osy`](tests/tent.test.osy).

```osy
using Osysharp.Identity;
using Osysharp.UserAccounts;
using Osysharp.ShiftScheduling;
using Osysharp.Qualifications;

[Test(Tent)]
[runas(Bo)]
void a_certificate_that_lapses_before_the_shift_refuses_it_in_the_qualifications_kits_words() {
  // It still holds today — what counts is the day of the shift.
  Assert.True(HasValidQualification(Bo, "first-aid"));
  Assert.Equal("First aid post needs a first aid certificate: First aid certificate expired on 5 July 2026.",
               ClaimShift(Shift.Single(s => s.Title == "Main stage post")).Message);
  runas(Cat) {
    RecordQualification(Bo, "first-aid");
    Assert.True(AssignShift(Shift.Single(s => s.Title == "Main stage post"), Bo).IsAssigned);
  }
}
```

From [`tests/tent.test.osy`](tests/tent.test.osy).

```osy
using Osysharp.Identity;
using Osysharp.UserAccounts;
using Osysharp.ShiftScheduling;
using Osysharp.Qualifications;

[Test(Tent)]
[runas(Bo)]
void a_doctor_on_the_festivals_own_register_meets_it_whatever_the_certificate_says() {
  runas(Cat) { new MedicalRegister { Doctor = Bo }; UnitOfWork.Commit(); }
  Assert.True(ClaimShift(Shift.Single(s => s.Title == "Main stage post")).IsAssigned);
}
```

From [`tests/tent.test.osy`](tests/tent.test.osy).

## Learn more

- [The Osysharp.ShiftScheduling.Qualifications reference](https://osysharp.com/reference/ui/qualifications-kit/)
- [Osysharp.ShiftScheduling.Qualifications in the package catalogue](https://osysharp.com/packages/#osysharp-shiftscheduling-qualifications)
- [The Osy# reference](https://osysharp.com/reference/) — the language, the runtime and every kit
- [`llms-full.txt`](https://osysharp.com/llms-full.txt) — the whole reference and specification as one Markdown file, for a coding agent

## Licence

MIT — see [`LICENSE`](LICENSE).
