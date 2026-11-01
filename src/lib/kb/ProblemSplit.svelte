<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { causeOf, uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  // Counted the way CauseTable counts: one-day moves stay under their tenant's cause, and their own row is not added on top.
  let { ticket, tickets }: { ticket: number; tickets: { parser: string; hint: string; oneDay: string } } = $props();

  const merged = $derived(uniqueById(reports as Report[]).filter((r) => r.mergedInto === ticket));
  const oneDay = $derived(merged.filter((r) => notes.oneDayMove.includes(r.id)));
  const rows = $derived([
    { problem: 'parser reads day-first dates month first', link: tickets.parser, n: merged.filter((r) => causeOf(r, tenantOf(r)) === 'parser').length },
    { problem: 'booking-v3 form hint says MM/DD', link: tickets.hint, n: merged.filter((r) => causeOf(r, tenantOf(r)) === 'hint').length },
    { problem: 'dates move by one day (inside the parser count)', link: tickets.oneDay, n: oneDay.length },
  ]);
  const total = $derived(merged.length);
</script>

<table>
  <thead><tr><th>problem</th><th>reports</th><th>ticket</th></tr></thead>
  <tbody>
    {#each rows as r (r.link)}
      <tr><td>{r.problem}</td><td>{r.n}</td><td><a href={r.link}>{r.link.split('/').slice(-3).join('/')}</a></td></tr>
    {/each}
  </tbody>
  <tfoot><tr><td>all reports in #{ticket}, each once</td><td>{total}</td><td></td></tr></tfoot>
</table>
