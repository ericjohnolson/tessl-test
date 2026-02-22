# Task: Refactor with Critic Mode

Refactor the following module. Use the refactor skill (`/refactor`).

## Code to Refactor

**File: `src/utils/dateFormatter.ts`**

```typescript
export function formatDate(d: Date, fmt: string): string {
  const yr = d.getFullYear();
  const mo = d.getMonth() + 1;
  const dy = d.getDate();
  const hr = d.getHours();
  const mn = d.getMinutes();
  const sc = d.getSeconds();

  let result = fmt;
  result = result.replace('YYYY', yr.toString());
  result = result.replace('MM', mo < 10 ? '0' + mo : mo.toString());
  result = result.replace('DD', dy < 10 ? '0' + dy : dy.toString());
  result = result.replace('HH', hr < 10 ? '0' + hr : hr.toString());
  result = result.replace('mm', mn < 10 ? '0' + mn : mn.toString());
  result = result.replace('ss', sc < 10 ? '0' + sc : sc.toString());

  return result;
}

export function isWeekend(d: Date): boolean {
  if (d.getDay() === 0) {
    return true;
  }
  if (d.getDay() === 6) {
    return true;
  }
  return false;
}

export function addDays(d: Date, n: number): Date {
  const result = new Date(d);
  result.setDate(result.getDate() + n);
  return result;
}

export function daysBetween(a: Date, b: Date): number {
  const ms = Math.abs(b.getTime() - a.getTime());
  return Math.floor(ms / (1000 * 60 * 60 * 24));
}
```

**File: `tests/utils/dateFormatter.test.ts`** (existing tests pass)

The code has both obvious improvements (variable names, simplify isWeekend, extract padZero helper) and subtler ones (repeated padding pattern, potential for a component map in formatDate, `replace` only replaces first occurrence which is a bug risk).

After the main refactoring pass, the agent should enter critic/final evaluation mode to catch anything missed.
