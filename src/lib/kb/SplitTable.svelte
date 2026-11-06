<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { groupByCause, uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  // One row per problem found in a merged ticket. Counts as CauseTable counts them; reports the desk
  // read as a one-day move (desk-notes.json) also get their own row: they are not a day/month swap.
  let { ticket, issues }: { ticket: number; issues: Record<string, string> } = $props();

  const merged = $derived(uniqueById(reports as Report[]).filter((r) => r.mergedInto === ticket));
  const shifted = $derived(merged.filter((r) => notes.oneDayMove.includes(r.id)));
  const rows = $derived([
    ...groupByCause(merged, tenantOf).map((g) => ({ problem: g.cause, n: g.reports.length, tenants: g.tenants })),
    { problem: 'one-day shift', n: shifted.length, tenants: new Set(shifted.map((r) => r.tenant)).size },
  ]);
</script>

<p><strong>{merged.length}</strong> reports merged into #{ticket} · <strong>{rows.length}</strong> problems</p>
<p class="note">The {shifted.length} one-day reports are filed under <code>parser</code> by tenant and counted there too.</p>

<table>
  <thead><tr><th>problem</th><th>reports</th><th>tenants</th><th>ticket</th></tr></thead>
  <tbody>
    {#each rows as r (r.problem)}
      <tr>
        <td><code>{r.problem}</code></td>
        <td>{r.n}</td>
        <td>{r.tenants}</td>
        <td>{#if issues[r.problem]}<a href={issues[r.problem]}>{issues[r.problem].split('/').slice(-3).join('/')}</a>{/if}</td>
      </tr>
    {/each}
  </tbody>
</table>
