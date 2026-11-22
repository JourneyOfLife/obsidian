---
tags: [dashboard, tasks]
---

# 📋 Tasks Dashboard

All open reminders across the vault, in one place. Requires the **Tasks** community plugin
(already enabled). These are *queries* — they auto-collect every `- [ ]` task you write
anywhere. Do not copy tasks here; write them where the work lives and they show up below.

> Add a task anywhere: `- [ ] Do the thing 🔁 due: 2026-11-25`
> Recurring: `🔁 every week`. Toggle done with `Ctrl/Cmd + click` on the checkbox.

---

## 🔴 Overdue
```tasks
not done
due before: today
sort by due
```

## 🟡 Due soon (next 7 days)
```tasks
not done
due after: yesterday
due before: next week
sort by due
```

## ⚪ No due date (triage these)
```tasks
not done
no due date
sort by path
```

## 🔥 High priority
```tasks
not done
priority is high
sort by due
```

## 🔁 Recurring
```tasks
not done
is recurring
sort by due
```

---

## Done recently
```tasks
done
done after: 30 days ago
sort by done
```
