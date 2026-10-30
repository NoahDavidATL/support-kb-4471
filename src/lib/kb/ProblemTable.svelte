<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import type { Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';
  import { problemRows, type Problem } from '#lib/kb/problems';

  type Ticket = { issue: number; label: string; state: string };

  let {
    ticket,
    repo,
    tickets,
  }: { ticket: number; repo: string; tickets: Partial<Record<Problem, Ticket>> } = $props();

  const result = $derived(problemRows(reports as Report[], ticket, tenantOf, notes.oneDayMove));
</script>

<p class="sum">
  <strong>{result.total}</strong> unique reports merged into #{ticket} ·
  <strong>{result.rows.length}</strong>
  {result.rows.length === 1 ? 'problem' : 'problems'}
</p>

<table>
  <thead>
    <tr><th>problem</th><th>reports</th><th>tenants</th><th>seen</th><th>ticket</th><th>state</th></tr>
  </thead>
  <tbody>
    {#each result.rows as row (row.problem)}
      {@const t = tickets[row.problem]}
      <tr>
        <td><code>{row.problem}</code> {t?.label ?? ''}</td>
        <td>{row.reports}</td>
        <td>{row.tenants}</td>
        <td>{row.firstSeen} → {row.lastSeen}</td>
        <td>{#if t}<a href="{repo}/issues/{t.issue}">#{t.issue}</a>{:else}none yet{/if}</td>
        <td class="state">{t?.state ?? 'open'}</td>
      </tr>
    {/each}
  </tbody>
</table>

{#each result.rows as row (row.problem)}
  {#if row.filedUnder}
    <p class="note">The {row.reports} <code>{row.problem}</code> reports are also counted under <code>{row.filedUnder}</code>, where the desk files them by tenant.</p>
  {/if}
{/each}

<style>
  .sum { font-size: 1.1rem; }
  .state { color: var(--muted); font-size: 0.85em; }
  .note { color: var(--muted); font-size: 0.85em; }
</style>
