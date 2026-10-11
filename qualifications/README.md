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
