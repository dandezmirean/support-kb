<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { groupByCause, readAs, uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  // Tickets filed out of `ticket`, counted the way CauseTable counts: the hint row, and the one-day notes.
  let { ticket, hint, oneDayMove }: { ticket: number; hint: string; oneDayMove: string } = $props();

  const merged = $derived(uniqueById(reports as Report[]).filter((r) => r.mergedInto === ticket));
  const groups = $derived(groupByCause(merged, tenantOf));
  const rows = $derived([
    { problem: 'hint', url: hint, count: groups.find((g) => g.cause === 'hint')?.reports.length ?? 0 },
    { problem: 'one-day move', url: oneDayMove, count: groups.reduce((n, g) => n + readAs(g, notes.oneDayMove), 0) },
  ]);
  const number = (url: string) => url.replace(/^.*\/issues\//, '#');
</script>

<table>
  <thead>
    <tr><th>ticket</th><th>problem</th><th>reports</th></tr>
  </thead>
  <tbody>
    {#each rows as r (r.problem)}
      <tr>
        <td><a href={r.url}>{number(r.url)}</a></td>
        <td>{r.problem}</td>
        <td>{r.count}</td>
      </tr>
    {/each}
  </tbody>
</table>
