# n8n Workflow Audit

**Auditor:** Jay (Squad Lead)
**Date:** 2026-02-16
**Instance:** https://n8n.srv1155599.hstgr.cloud
**Status:** COMPLETE

---

## Summary

| Category | Count |
|----------|-------|
| Total Workflows | 50 |
| Active | 4 |
| Inactive | 46 |
| Recommend Keep | 8 |
| Recommend Delete | 38 |
| Recommend Activate | 4 |

---

## Active Workflows (4)

| ID | Name | Last Updated | Status |
|----|------|--------------|--------|
| XwoxMVmJtoTlM53S | 🔧 Parts Search Agent V3.1.1 - Production | 2025-12-20 | ✅ Active |
| tyQpxDUxZJmMHB0h | SMS Broadcast - RevenueBooster v2 | 2025-12-17 | ✅ Active |
| hXzBp4eqrHrK86YA | Daily Email Scan KYS Havana | 2025-12-15 | ✅ Active |
| VsYcid3DaTyv1V3K | Iron Secretary - Voice Note Digest Add-on | 2025-12-09 | ✅ Active |

**Assessment:** All 4 active workflows appear to be production workloads. Verify they're still running correctly.

---

## Inactive Workflows — Categorized

### 🟢 KEEP (8) — Business Value, May Reactivate

| ID | Name | Last Updated | Reason |
|----|------|--------------|--------|
| KM0MSoOXp7VAsVCH | ReviewGuard Sales Machine v2 | 2026-02-16 | Recently updated, active development |
| C687mvO0AiPdIyl2 | ReviewGuard Sales Machine v2 (Clean) | 2026-02-04 | Backup/clean version |
| IEcKG3Z4PTnyqpp7 | Google Review Auto-Reply | 2025-12-18 | Core ReviewGuard feature |
| Mjj5i3uDQonB48NG | Missed Call Auto-Reply (ReviewGuard) | 2025-12-14 | Core ReviewGuard feature |
| H8egtwZGN9AbS3PY | Iron Secretary - Main V2 | 2025-12-13 | May need for Iron Secretary |
| 75U4KQP1zoYTrYEY | Iron Secretary - Main Old | 2025-12-12 | Backup reference |
| uk8rmovPrHw3qZ2V | Query Data Workflow | 2025-12-17 | Utility workflow |
| IgZKuH2vB2wRPLjD | n8n Developer Agent | 2025-12-15 | Development utility |

### 🟡 CONSIDER ACTIVATING (4)

| ID | Name | Reason |
|----|------|--------|
| KM0MSoOXp7VAsVCH | ReviewGuard Sales Machine v2 | Recently updated — ready for production? |
| IEcKG3Z4PTnyqpp7 | Google Review Auto-Reply | Core feature — should this be active? |
| Mjj5i3uDQonB48NG | Missed Call Auto-Reply | Core feature — should this be active? |
| H8egtwZGN9AbS3PY | Iron Secretary - Main V2 | Superseded by Supabase Edge Functions? |

### 🔴 DELETE (38) — Obsolete, Duplicates, or Abandoned

#### Hamoriko Video Pipeline (20 workflows)
All Hamoriko workflows are content/video generation pipeline — appears to be abandoned project.

| ID | Name |
|----|------|
| K7VaeE3AMzjRZxCG | Hamoriko_Master_Orchestrator_v4 |
| lXQKF79jULPoJJng | Hamoriko_Publish_Manager_v4 |
| 4y6dl209UfxdeleO | Hamoriko_Assembly_Editor_v4 |
| d1YUNYpMdUCHRaze | Hamoriko_Voice_Studio_v4 |
| 27OeglxDKOA24SKw | Hamoriko_Visual_Director_v4 |
| bXKfbYl4dddedKkG | Hamoriko_Safety_Gate_v4 |
| bx2Dfq4QTndpTlL0 | Hamoriko_Story_Engine_v4 |
| cGvZ20CbqJzV0xQu | Hamoriko_Error_Handler_v4 |
| 88YUlvFf7akwNZTv | hamoriko_lora_image_generator_v1.0 |
| ZaFl5Rf1k8kZZVJS | Content_Safety_v4 (Enhanced Child Protection) |
| GoxnqRwoubnUnwT7 | Visual_Generation_v1.4_Flux (Micro-Polished) |
| 2gCPjQiMEeH5CTZ5 | Voice_Generation_v5.3 (Production Ready) |
| JVyGKNC2lxIdEf6C | Hamoriko_Master_v5.5 (Micro-Polished) |
| cW2qR9oaDWAGeRWK | Hamoriko Master v5.2 |
| ccC4EUcAYlerdMEC | Hamoriko Story Generator - Master |
| 9QJEEkehUTnp3E86 | Hamoriko_Master_Orchestrator_v4 (duplicate) |
| qLfQTHGn7RQWcaQh | Hamoriko_Publish_Manager_v4 (duplicate) |
| eVRlQ4hHyr0i4Et4 | Hamoriko_Assembly_Editor_v4 (duplicate) |
| enjZZqoiDoEKJ6wf | Hamoriko Asset Generator - Phase 2 |
| U3MsW27zK6kc1QiV | Hamoriko_Story_Engine_v4 (duplicate) |
| vNqwAUzYBUQui6UE | Hamoriko_Visual_Director_v4 (duplicate) |
| V3zOYzu6rWUgBdBu | Hamoriko_Safety_v4 (duplicate) |
| AXJdcb91aJa9LOol | Hamoriko_Errors_v4 (duplicate) |
| AeyPkkZHWy5LU0sS | Hamoriko_Voice_v4 (duplicate) |
| iz0LsIoBXda1Y6YH | R40 Hyperconsistent Storymaker template |

#### Voice/Media Generation (5 workflows)
| ID | Name |
|----|------|
| hY2xuZBXdDJoad7K | Create Voice |
| Q7RtWo337rs5UIGL | Combine Clips and Subs |
| 9AlVCy0kGyUNmAch | Create Video |
| 1FynkyCOKXmbAlEV | Create Image |
| XInEwrFA2CHnqio7 | Voice Generation v5.1 |
| xLTANg5GtYCXBRBA | Voice Generation v5.1 (duplicate) |
| 8ppuXlwgMWVIkZ9P | Voice Generation v5 |

#### Test/Empty/Abandoned (6 workflows)
| ID | Name |
|----|------|
| DozzOmJ5j0vByS6J | Empty Workflow - No Request Provided |
| YWO4uhcnvfQkZYD9 | n8n Developer Agent (Fixed) |
| JzjGkljCyySVPQpd | Query Handler Workflow |
| bU429lLsvudKtkX2 | Daily Weather to Slack |
| QU1Z98NuHk9qrEhe | Safety Check |
| hosc5S5AOWSc3yYM | My workflow |

---

## Recommendations

### Immediate Actions

1. **Verify Active Workflows**
   - Check execution logs for Parts Search Agent
   - Confirm SMS Broadcast is sending correctly
   - Verify Daily Email Scan is processing

2. **Delete Hamoriko Workflows**
   - 20+ abandoned video pipeline workflows
   - Cluttering the workspace
   - No business value for current operations

3. **Review ReviewGuard Workflows**
   - Sales Machine v2 was updated TODAY (2026-02-16) — is this ready to activate?
   - Google Review Auto-Reply and Missed Call Auto-Reply seem like core features — why inactive?

### Consolidation Opportunities

1. **Iron Secretary:** 
   - 2 inactive n8n workflows + 1 active add-on
   - Iron Secretary V2 moved to Supabase Edge Functions
   - Consider deleting n8n versions if Edge Functions are production

2. **Voice Generation:**
   - Multiple versions (v5, v5.1, v5.3)
   - Keep only latest if needed, delete rest

### New Workflows to Consider

| Workflow | Purpose | Priority |
|----------|---------|----------|
| Scout Deal Alerts | Backup to Vercel cron | LOW (Vercel handles this) |
| ReviewGuard Onboarding | Automate new customer setup | MEDIUM |
| Daily Digest | Consolidate alerts from all agents | MEDIUM |

---

## Cleanup Script

To delete obsolete workflows (run after Shuki approval):

```bash
# Hamoriko workflows (20)
for id in K7VaeE3AMzjRZxCG lXQKF79jULPoJJng 4y6dl209UfxdeleO d1YUNYpMdUCHRaze 27OeglxDKOA24SKw bXKfbYl4dddedKkG bx2Dfq4QTndpTlL0 cGvZ20CbqJzV0xQu 88YUlvFf7akwNZTv ZaFl5Rf1k8kZZVJS GoxnqRwoubnUnwT7 2gCPjQiMEeH5CTZ5 JVyGKNC2lxIdEf6C cW2qR9oaDWAGeRWK ccC4EUcAYlerdMEC 9QJEEkehUTnp3E86 qLfQTHGn7RQWcaQh eVRlQ4hHyr0i4Et4 enjZZqoiDoEKJ6wf U3MsW27zK6kc1QiV vNqwAUzYBUQui6UE V3zOYzu6rWUgBdBu AXJdcb91aJa9LOol AeyPkkZHWy5LU0sS iz0LsIoBXda1Y6YH; do
  curl -X DELETE "https://n8n.srv1155599.hstgr.cloud/api/v1/workflows/$id" \
    -H "X-N8N-API-KEY: $N8N_API_KEY"
done
```

⚠️ **Do not run without Shuki approval.** Deletions are irreversible.

---

## Summary

- **4 active workflows** — verify functioning
- **8 to keep** — business value
- **38 to delete** — obsolete Hamoriko project + duplicates
- **4 to consider activating** — ReviewGuard features

After cleanup, n8n will have ~12 workflows instead of 50 — much cleaner.

---

*Report generated by Jay | Squad Lead | T1 (Execute)*
