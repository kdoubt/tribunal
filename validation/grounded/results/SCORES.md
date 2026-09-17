```
decision | arm | judge | decision_correct | must_catch | false_obj | grounding | support
g01 | S_claude | codex | 0.5 | 0.67 (4/6) | 2 | 1.00 | 0.54
g01 | S_codex | grok | 0.5 | 0.67 (4/6) | 2 | 1.00 | 0.42
g01 | S_grok | codex | 1 | 0.83 (5/6) | 0 | 1.00 | 0.79
g01 | A_2seat | codex | 0.5 (sens 0.5) | 0.83 | 2 | - | -  [derived from solos: dc 0 mc 0.83]
g01 | A_panel | codex | 0.5 | 0.50 (3/6) | 1 | - | -
g02 | S_claude | grok | 0.5 | 0.86 (6/7) | 3 | 1.00 | 0.57
g02 | S_codex | grok | 0.5 | 0.71 (5/7) | 3 | 1.00 | 0.69
g02 | S_grok | claude | 1 | 1.00 (7/7) | 0 | 1.00 | 0.97
g02 | A_2seat | grok | 0.5 (sens 0.5) | 1.00 | 3 | - | -  [derived from solos: dc 0.5 mc 1.00]
g02 | A_panel | grok | 0.5 | 0.86 (6/7) | 0 | - | -
g03 | S_claude | codex | 0 | 0.43 (3/7) | 0 | 1.00 | 0.33
g03 | S_codex | claude | 0.5 | 0.43 (3/7) | 3 | 0.62 | 0.83
g03 | S_grok | claude | 0.5 | 0.57 (4/7) | 4 | 1.00 | 0.95
g03 | A_2seat | claude | 0.5 (sens 0.5) | 0.71 | 4 | - | -  [derived from solos: dc 0.5 mc 0.71]
g03 | A_panel | claude | 0.5 | 0.57 (4/7) | 4 | - | -
g04 | S_claude | codex | 0.5 | 0.71 (5/7) | 1 | 0.75 | 0.69
g04 | S_codex | claude | 0.5 | 0.86 (6/7) | 2 | 0.00 | -
g04 | S_grok | claude | 1 | 0.86 (6/7) | 1 | 1.00 | 0.95
g04 | A_2seat | claude | 1 (sens 0.5) | 1.00 | 3 | - | -  [derived from solos: dc 0 mc 1.00]
g04 | A_panel | claude | 1 | 0.71 (5/7) | 0 | - | -
g05 | S_claude | codex | 1 | 0.86 (6/7) | 0 | 1.00 | 0.52
g05 | S_codex | grok | 1 | 1.00 (7/7) | 0 | 1.00 | 0.44
g05 | S_grok | codex | 1 | 0.86 (6/7) | 1 | 1.00 | 0.38
g05 | A_2seat | codex | 1 (sens 1) | 1.00 | 0 | - | -  [derived from solos: dc 1 mc 1.00]
g05 | A_panel | codex | 1 | 0.71 (5/7) | 1 | - | -
g06 | S_claude | grok | 1 | 0.83 (5/6) | 2 | 1.00 | 0.89
g06 | S_codex | grok | 1 | 1.00 (6/6) | 0 | 1.00 | 0.80
g06 | S_grok | claude | 1 | 1.00 (6/6) | 0 | 0.88 | 1.00
g06 | A_2seat | grok | 1 (sens 1) | 1.00 | 2 | - | -  [derived from solos: dc 1 mc 1.00]
g06 | A_panel | grok | 0.5 | 0.83 (5/6) | 2 | - | -
g07 | S_claude | grok | 0.5 | 0.71 (5/7) | 4 | 1.00 | 0.55
g07 | S_codex | claude | 0.5 | 0.86 (6/7) | 3 | 1.00 | 0.93
g07 | S_grok | claude | 1 | 0.86 (6/7) | 0 | 1.00 | 0.93
g07 | A_2seat | claude | 1 (sens 0.5) | 1.00 | 3 | - | -  [derived from solos: dc 0 mc 1.00]
g07 | A_panel | claude | 0.5 | 0.71 (5/7) | 0 | - | -
g08 | S_claude | codex | 0.5 | 0.25 (2/8) | 3 | 1.00 | 0.00
g08 | S_codex | claude | 1 | 0.88 (7/8) | 3 | 0.88 | 1.00
g08 | S_grok | claude | 1 | 0.75 (6/8) | 2 | 1.00 | 0.85
g08 | A_2seat | claude | 1 (sens 1) | 1.00 | 2 | - | -  [derived from solos: dc 1 mc 0.88]
g08 | A_panel | claude | 1 | 0.75 (6/8) | 0 | - | -
g09 | S_claude | codex | 0.5 | 0.27 (3/11) | 3 | 1.00 | 0.44
g09 | S_codex | grok | 0.5 | 0.73 (8/11) | 1 | 1.00 | 0.22
g09 | S_grok | codex | 0.5 | 0.27 (3/11) | 3 | 0.75 | 0.46
g09 | A_2seat | codex | 0.5 (sens 0.5) | 0.27 | 4 | - | -  [derived from solos: dc 0.5 mc 0.27]
g09 | A_panel | codex | 0.5 | 0.36 (4/11) | 1 | - | -
g10 | S_claude | codex | 0.5 | 0.71 (5/7) | 3 | n/a | -
g10 | S_codex | claude | 0.5 | 0.71 (5/7) | 2 | n/a | -
g10 | S_grok | claude | 0.5 | 0.71 (5/7) | 2 | n/a | -
g10 | A_2seat | claude | 0.5 (sens 0.5) | 0.71 | 3 | - | -  [derived from solos: dc 0.5 mc 0.71]
g10 | A_panel | claude | 1 | 0.71 (5/7) | 2 | - | -

== Primary endpoint (paired A_panel - A_2seat) ==
decisions paired: ['g01', 'g02', 'g03', 'g04', 'g05', 'g06', 'g07', 'g08', 'g09', 'g10']
must_catch_rate: mean diff -0.18 (n=10)
decision_correct (strict): mean diff -0.06 (n=9); sensitivity: +0.06
false_objections: panel mean 1.10 vs 2seat upper-bound mean 2.60
missing decisions: 0 (within cap)
```
