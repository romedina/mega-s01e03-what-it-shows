<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import type { Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';
  import { PROBLEMS, splitTicket, type Problem } from '#lib/kb/split';

  // issues: where each problem is tracked now, e.g. { parser: 'https://github.com/…/issues/1' }
  let { ticket, issues = {} }: { ticket: number; issues?: Partial<Record<Problem, string>> } = $props();

  const split = $derived(splitTicket(reports as Report[], ticket, tenantOf, notes.oneDayMove));
  const channels = (c: Record<string, number>) =>
    Object.entries(c)
      .sort((a, b) => b[1] - a[1])
      .map(([ch, n]) => `${ch} ×${n}`)
      .join(' · ');
</script>

<p class="sum">
  <strong>{split.unique}</strong> reports merged into #{ticket}
  ({split.rows} rows in the export, {split.rows - split.unique} of them repeats) ·
  <strong>{split.groups.length}</strong> {split.groups.length === 1 ? 'problem' : 'problems'}
</p>

<table>
  <thead>
    <tr><th>problem</th><th>reports</th><th>tenants</th><th>channels</th><th>intake tag wrong</th><th>seen</th><th>ticket</th></tr>
  </thead>
  <tbody>
    {#each split.groups as g (g.problem)}
      <tr>
        <td><code>{g.problem}</code><br /><span class="what">{PROBLEMS[g.problem]}</span></td>
        <td><strong>{g.reports.length}</strong></td>
        <td>{g.tenants.length}</td>
        <td class="small">{channels(g.channels)}</td>
        <td>{g.mistagged}</td>
        <td class="small">{g.firstSeen} → {g.lastSeen}</td>
        <td>{#if issues[g.problem]}<a href={issues[g.problem]}>{issues[g.problem]?.replace(/.*\/issues\//, 'issue #')}</a>{:else}<span class="none">none yet</span>{/if}</td>
      </tr>
    {/each}
  </tbody>
</table>

<style>
  .sum { font-size: 1.1rem; }
  .what, .small { color: var(--muted); font-size: 0.85em; }
  .none { color: var(--warn); font-weight: 600; }
</style>
