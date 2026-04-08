## Concepts
```dataview
table file.name as "Concept"
from "01_Concepts"
where topic = this.file.name
```

---

## Setups
```dataview
table file.name as "Setup"
from "04_Setups"
where topic = this.file.name
```

---

## Cheatsheets
```dataview
table file.name as "Cheatsheet"
from "02_Cheatsheets"
where topic = this.file.name
```