# Claude Code Prompt: Scout Intelligence System Phase 1

Copy this entire prompt and paste into Claude Code. It will generate the complete Scout filtering logic + UI enhancements for Mission Control dashboard.

---

## MISSION: Enhance Mission Control Dashboard with Scout Intelligence Filtering

**Context:** You are building Phase 1 enhancements to an agent coordination dashboard (https://github.com/shukik73/mission-control). The dashboard currently shows a basic Kanban workflow. We need to add Scout deal intelligence with advanced filtering, ROI calculations, and stream separation.

**Deliverable:** Updated `index.html` with all features below integrated. The file should be production-ready to deploy immediately.

---

## FEATURE 1: Scout Deal Card Component

Replace the generic task cards in SCOUT INBOX with detailed deal cards containing:

```
┌─────────────────────────────────────────┐
│ 🎯 [DEVICE MODEL]                       │
│ [Platform] • [Time Left] • [Location]   │
│                                         │
│ Cost: $XX  Shipping: $X  = $XX Total   │
│ Value: $XXX (30-day avg)                │
│                                         │
│ ROI: [COLOR-CODED]% | Profit: $XX       │
│                                         │
│ ✓ Seller 98% (2.4k sales)               │
│ ✓/⚠️ [Flags: Local pickup, Auction soon]│
│                                         │
│ [👍 Approve] [👎 Pass] [💬 Ask Jay]    │
└─────────────────────────────────────────┘
```

**Requirements:**
- ROI color-coded: 🟢 green (>100%), 🟡 yellow (50-100%), 🔴 red (<50%)
- Show all metadata: cost breakdown, seller validation, flags
- Buttons are clickable (href links for now; will integrate with Supabase later)

---

## FEATURE 2: Scout Auto-Pass Logic

Implement filtering rules that automatically hide low-quality deals from inbox:

```javascript
const SCOUT_FILTERS = {
  AUTO_PASS: {
    roi_threshold: 20,           // ROI < 20% auto-pass
    shipping_cost_ratio: 0.30,   // Shipping > 30% of price auto-pass
    seller_rating_min: 90,       // Seller rating < 90% auto-pass
    exclude_categories: ['furniture', 'clothing', 'home-appliances'], // Non-electronics auto-pass
    exceptions: {
      // Exception: Non-electronics with ROI > 100% show in inbox (for Jay review)
      high_roi_non_electronics: 100
    }
  },
  INBOX: {
    show_if: [
      'roi >= 50',
      'roi >= 20 && (local_pickup || rare_part || bulk_lot)',
      'seller_rating >= 90 && roi >= 20'
    ]
  }
};
```

**Implementation:**
- Create function `shouldShowDeal(deal)` that returns true/false based on filters
- Log all passed deals to `trends_database` (simulate with localStorage for now) with reason
- Display auto-pass decisions in activity feed: "⏭️ Passed: Nintendo Switch (18% ROI < 20% threshold)"

---

## FEATURE 3: Local Pickup Economics Calculator

When a deal has local pickup available, calculate true ROI including:
- Gas cost: `(distance_miles / 25) * 3.50` (25 mpg, $3.50/gal)
- Time cost: `(distance_miles / 30) * 50` ($50/hr opportunity cost)

```javascript
function calculateTrueROI(deal) {
  let totalCost = deal.price + deal.shipping_cost;
  
  if (deal.is_local_pickup && deal.distance_miles < 25) {
    const gasCost = (deal.distance_miles / 25) * 3.50;
    const timeCost = (deal.distance_miles / 30) * 50;
    const pickupCost = gasCost + timeCost;
    
    deal.pickup_economics = {
      distance: deal.distance_miles,
      gas_cost: gasCost.toFixed(2),
      time_cost: timeCost.toFixed(2),
      total_pickup_cost: pickupCost.toFixed(2),
      roi_with_pickup: (((deal.estimated_value - totalCost - pickupCost) / (totalCost + pickupCost)) * 100).toFixed(1)
    };
    
    totalCost += pickupCost;
  }
  
  return ((deal.estimated_value - totalCost) / totalCost * 100).toFixed(1);
}
```

**Display in card:**
```
Cost: $89 + $0 ship = $89
Pickup: $23 (17mi: $8 gas + $15 time)
Total: $112
Value: $180
ROI: 61% (with pickup factored in) ✓ Still worth it
```

---

## FEATURE 4: Urgent Auction Countdown

For auction listings ending in < 1 hour with ROI > 50%:
- Add pulsing red border to card
- Show countdown in red: "⏱️ ENDING IN 42m"
- Visual urgency indicator

```javascript
function getAuctionStatus(auction_ends_at) {
  if (!auction_ends_at) return null;
  
  const now = Date.now();
  const endsAt = new Date(auction_ends_at).getTime();
  const hoursLeft = (endsAt - now) / (1000 * 60 * 60);
  
  if (hoursLeft < 1) {
    const minutesLeft = Math.round(hoursLeft * 60);
    return {
      urgent: true,
      status: `ENDING IN ${minutesLeft}m`,
      borderClass: 'pulsing-red'
    };
  }
  return { urgent: false };
}
```

**CSS for pulsing effect:**
```css
.card.pulsing-red {
  border: 2px solid #ef4444;
  animation: pulse-red 1.5s ease-in-out infinite;
}
@keyframes pulse-red {
  0%, 100% { border-color: #ef4444; box-shadow: 0 0 0 0 rgba(239, 68, 68, 0.7); }
  50% { border-color: #dc2626; box-shadow: 0 0 0 8px rgba(239, 68, 68, 0); }
}
```

---

## FEATURE 5: Category Validation for Non-Electronics

If item is not electronics:
- ROI < 100%: Auto-pass silently
- ROI > 100%: Show in inbox with "⚠️ CATEGORY ALERT" badge + question: "Flip for cash or outside expertise?"

```javascript
function validateCategory(deal) {
  const electronicsCategories = ['laptop', 'phone', 'ipad', 'tv', 'logic board', 'screen', 'camera'];
  const isElectronics = electronicsCategories.some(cat => 
    deal.category.toLowerCase().includes(cat) || 
    deal.title.toLowerCase().includes(cat)
  );
  
  if (!isElectronics && deal.roi < 100) {
    return { auto_pass: true, reason: 'non_electronics_low_roi' };
  }
  
  if (!isElectronics && deal.roi >= 100) {
    return { 
      show: true, 
      flag: 'CATEGORY ALERT',
      question: 'Flip for cash or outside our expertise?'
    };
  }
  
  return { show: true };
}
```

---

## FEATURE 6: Stream Toggle (Deals vs SaaS Ops)

Add filter buttons at the top of the dashboard:

```html
<div class="stream-selector">
  <button class="stream-btn active" data-stream="deals">
    💰 Scout Deals (3)
  </button>
  <button class="stream-btn" data-stream="saas">
    🚀 SaaS Ops (5)
  </button>
  <button class="stream-btn" data-stream="all">
    📊 All Streams (8)
  </button>
</div>
```

**Behavior:**
- Deals mode: Show only [SCOUT INBOX, NEEDS SHUKI (deals), DONE (deals)]
- SaaS mode: Show only [SaaS INBOX, NEEDS SHUKI (saas), DONE (saas)]
- All mode: Show all columns
- Persist selection in localStorage

**Implementation:**
```javascript
function filterByStream(stream) {
  const dealsColumns = ['scout-inbox', 'needs-shuki-deals', 'done-deals'];
  const saasColumns = ['saas-inbox', 'needs-shuki-saas', 'done-saas'];
  
  const allColumns = document.querySelectorAll('.kanban-column');
  
  if (stream === 'deals') {
    allColumns.forEach(col => {
      col.style.display = dealsColumns.includes(col.id) ? 'flex' : 'none';
    });
  } else if (stream === 'saas') {
    allColumns.forEach(col => {
      col.style.display = saasColumns.includes(col.id) ? 'flex' : 'none';
    });
  } else {
    allColumns.forEach(col => col.style.display = 'flex');
  }
  
  localStorage.setItem('activeStream', stream);
}

// On page load, restore last selected stream
window.addEventListener('DOMContentLoaded', () => {
  const saved = localStorage.getItem('activeStream') || 'all';
  filterByStream(saved);
  document.querySelector(`[data-stream="${saved}"]`).classList.add('active');
});
```

---

## FEATURE 7: Activity Feed with Auto-Pass Logging

Update the right panel activity feed to show:
- ✓ Deals approved
- ⏭️ Deals auto-passed with reason
- ⚠️ Urgent auctions
- 📊 Trends (price moves, market shifts)

Example entries:
```
✓ Approved: MacBook Pro A1502 (102% ROI)
⏭️ Passed: Nintendo Switch (18% ROI < 20%)
⏱️ Urgent: Bulk M2 boards (45% ROI, 38 min left) 
⚠️ Category: Rolex box (125% ROI) — Outside expertise?
```

Store in localStorage `trends_db`:
```javascript
const trendsDB = {
  weekly_stats: {
    deals_found: 127,
    deals_approved: 23,
    deals_passed: 104,
    pass_reasons: {
      roi_too_low: 67,
      shipping_expensive: 21,
      seller_sketchy: 12,
      non_electronics: 4
    }
  }
};
```

---

## TEST DATA

Include sample deals in HTML to test all features:

```javascript
const SAMPLE_DEALS = [
  {
    id: 1,
    title: "MacBook Pro A1502 Logic Board 8GB",
    platform: "eBay",
    category: "logic-board",
    price: 89,
    shipping_cost: 0,
    estimated_value: 180,
    location: "Miami, FL",
    distance_miles: 3,
    is_local_pickup: true,
    seller_rating: 98,
    seller_transactions: 2400,
    auction_ends_at: new Date(Date.now() + 2 * 60 * 60 * 1000), // 2 hours
    roi: 102,
    flags: ["Local pickup available", "Seller 98% (2.4k sales)"]
  },
  {
    id: 2,
    title: "Sony WH-1000XM4 Wireless Headphones - Black",
    platform: "Mercari",
    category: "electronics",
    price: 128,
    shipping_cost: 15,
    estimated_value: 240,
    location: "Austin, TX",
    distance_miles: 1200,
    is_local_pickup: false,
    seller_rating: 100,
    seller_transactions: 54,
    roi: 78,
    flags: ["Seller 100% (54 sales)"]
  },
  {
    id: 3,
    title: "Nintendo Switch Lite (Coral) - Console Only",
    platform: "OfferUp",
    category: "gaming",
    price: 110,
    shipping_cost: 0,
    estimated_value: 130,
    location: "Los Angeles, CA",
    distance_miles: 2000,
    is_local_pickup: false,
    seller_rating: 85,
    seller_transactions: 12,
    roi: 18,
    flags: ["Seller 85% (low rating)", "Non-electronics"]
  },
  {
    id: 4,
    title: "Rolex Submariner Box & Papers Only",
    platform: "eBay",
    category: "luxury-accessories",
    price: 350,
    shipping_cost: 20,
    estimated_value: 800,
    location: "Chicago, IL",
    distance_miles: 1500,
    is_local_pickup: false,
    seller_rating: 92,
    seller_transactions: 300,
    roi: 116,
    flags: ["Non-electronics", "Category alert: Outside expertise?"]
  },
  {
    id: 5,
    title: "Bulk Lot: 5x iPhone 13 Logic Boards",
    platform: "eBay",
    category: "logic-boards",
    price: 950,
    shipping_cost: 35,
    estimated_value: 2100,
    location: "New York, NY",
    distance_miles: 800,
    is_local_pickup: false,
    seller_rating: 99,
    seller_transactions: 5000,
    auction_ends_at: new Date(Date.now() + 30 * 60 * 1000), // 30 minutes
    roi: 121,
    flags: ["Bulk lot (5 units)", "Urgent: 30 min left", "Seller 99% (5k sales)"]
  }
];
```

---

## REQUIREMENTS

1. **No external dependencies** — Pure HTML/CSS/JS only
2. **All features integrated** — Every sample deal tests a different filter/feature
3. **Responsive design** — Works on desktop and tablet
4. **Production-ready** — Clean code, no console errors
5. **localStorage** — Persist stream selection + trends database
6. **Accessibility** — Proper color contrast, keyboard navigation

---

## OUTPUT

Return a complete, working `index.html` file that:
- ✅ Includes all 7 features above
- ✅ Renders sample deals correctly
- ✅ Filters work (auto-pass, stream toggle, category validation)
- ✅ Activity feed shows auto-pass reasons
- ✅ Ready to push to GitHub and deploy immediately

---

**That's it. Build Scout Phase 1 with all critical improvements integrated.**
