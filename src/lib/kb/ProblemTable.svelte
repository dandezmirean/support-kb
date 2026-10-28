<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { groupByCause, uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  // One row per problem: the causes from causes.ts, with the desk's one-day moves taken out as their own row.
  let { ticket, issues }: { ticket: number; issues: Record<string, string> } = $props();

  const merged = $derived(uniqueById(reports as Report[]).filter((r) => r.mergedInto === ticket));
  const moved = $derived(merged.filter((r) => notes.oneDayMove.includes(r.id)));
  const rows = $derived(
    [
      ...groupByCause(merged.filter((r) => !moved.includes(r)), tenantOf).map((g) => ({ problem: g.cause as string, reports: g.reports })),
      { problem: 'oneDayMove', reports: moved },
    ].map(({ problem, reports }) => {
      const days = reports.map((r) => r.receivedAt.slice(0, 10)).sort();
      return { problem, count: reports.length, tenants: new Set(reports.map((r) => r.tenant)).size, firstSeen: days[0], lastSeen: days[days.length - 1] };
    }),
  );
</script>

<p class="sum">
  <strong>{merged.length}</strong> reports merged into #{ticket} · <strong>{rows.length}</strong> problems
</p>

<table>
  <thead>
    <tr><th>problem</th><th>reports</th><th>tenants</th><th>seen</th><th>ticket</th></tr>
  </thead>
  <tbody>
    {#each rows as r (r.problem)}
      <tr>
        <td><code>{r.problem}</code></td>
        <td>{r.count}</td>
        <td>{r.tenants}</td>
        <td>{r.firstSeen} → {r.lastSeen}</td>
        <td>{#if issues[r.problem]}<a href={issues[r.problem]}>{issues[r.problem].replace(/^.*\/issues\//, '#')}</a>{/if}</td>
      </tr>
    {/each}
  </tbody>
</table>

<style>
  .sum { font-size: 1.1rem; }
</style>
