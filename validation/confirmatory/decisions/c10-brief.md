# Decision C10 - storing times for future recurring events

A calendar feature lets users schedule **recurring** meetings ("every Monday 09:00
America/New_York"). An engineer proposes: compute each occurrence's UTC instant at
creation time and store those UTC timestamps; render by converting UTC->local.
**Question:** is "convert to UTC at creation and store UTC" correct for these
future recurring local-time events? Commit and name the single biggest risk.
