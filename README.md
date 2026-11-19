import React, { useState, useMemo } from 'react';

// --- BRESONA EVENTS INITIAL CATALOG ---
const INITIAL_INVENTORY = [
  {
    id: 'BRS-001',
    name: 'Celebration Gold Chiavari Chair',
    category: 'Seating',
    rate: 8.5,
    totalStock: 250,
    damagedStock: 5,
    image: '🪑',
    description: 'Lightweight, festive gold chiavari chair with soft ivory cushion for events & galas.',
  },
  {
    id: 'BRS-002',
    name: 'Statement Hexagon Backdrop Arch (2.4m)',
    category: 'Backdrops',
    rate: 95.0,
    totalStock: 6,
    damagedStock: 0,
    image: '⬡',
    description: 'Eye-catching geometric arch frame ready for balloon garlands, florals, or photo booths.',
  },
  {
    id: 'BRS-003',
    name: 'Warm Ivory Satin Table Runner',
    category: 'Linens',
    rate: 6.0,
    totalStock: 180,
    damagedStock: 8,
    image: '🧣',
    description: 'Smooth, stain-resistant table runner that pairs effortlessly with bright or classic themes.',
  },
  {
    id: 'BRS-004',
    name: 'Party & Stage RGBWA LED Uplight',
    category: 'Lighting',
    rate: 25.0,
    totalStock: 48,
    damagedStock: 2,
    image: '💡',
    description: 'Cordless high-output color uplight with 12-hour battery for receptions and stages.',
  },
  {
    id: 'BRS-005',
    name: 'Crystal 5-Arm Centerpiece Stand',
    category: 'Centerpieces',
    rate: 35.0,
    totalStock: 30,
    damagedStock: 1,
    image: '🕯️',
    description: 'Sparkling multi-arm table centerpiece that catches light across banquet halls.',
  },
  {
    id: 'BRS-006',
    name: 'Round Banquet Table (180cm / 10-Seater)',
    category: 'Tables',
    rate: 18.0,
    totalStock: 40,
    damagedStock: 0,
    image: '⭕',
    description: 'Sturdy 10-guest round folding banquet table built for fast setup and breakdown.',
  },
];

const INITIAL_ORDERS = [
  {
    id: 'BRS-ORD-101',
    clientName: 'Selamawit Tadesse',
    eventType: 'Wedding Celebration',
    startDate: '2026-11-28',
    endDate: '2026-11-30',
    venue: 'Skylight Hotel Ballroom',
    status: 'CONFIRMED',
    items: [
      { itemId: 'BRS-001', qty: 150, rate: 8.5 },
      { itemId: 'BRS-002', qty: 2, rate: 95.0 },
    ],
  },
  {
    id: 'BRS-ORD-102',
    clientName: 'Horizon Tech Summit',
    eventType: 'Corporate Event',
    startDate: '2026-12-03',
    endDate: '2026-12-04',
    venue: 'Millennium Hall',
    status: 'DISPATCHED',
    items: [
      { itemId: 'BRS-004', qty: 30, rate: 25.0 },
      { itemId: 'BRS-006', qty: 20, rate: 18.0 },
    ],
  },
];

// Signature winking-smiley icon inspired by the BRESONA logo 'O'
function BresonaWinkIcon({ className = 'w-6 h-6' }) {
  return (
    <svg viewBox="0 0 64 64" className={className} fill="none" aria-hidden="true">
      <defs>
        <linearGradient id="bresonaGoldGrad" x1="0" y1="0" x2="1" y2="1">
          <stop offset="0%" stopColor="#FFD21E" />
          <stop offset="100%" stopColor="#FF8800" />
        </linearGradient>
      </defs>
      <circle cx="32" cy="32" r="30" fill="url(#bresonaGoldGrad)" />
      {/* Left open eye */}
      <circle cx="22" cy="25" r="3.5" fill="#FFFFFF" />
      {/* Right winking eye */}
      <path
        d="M36 21L45 25L36 29"
        stroke="#FFFFFF"
        strokeWidth="3.2"
        strokeLinecap="round"
        strokeLinejoin="round"
      />
      {/* Joyful smile */}
      <path
        d="M20 36C22.5 45.5 41.5 45.5 44 36H20Z"
        fill="#FFFFFF"
      />
    </svg>
  );
}

export default function BresonaEventsApp() {
  const [viewMode, setViewMode] = useState('storefront'); // 'storefront' | 'admin'
  const [adminTab, setAdminTab] = useState('orders'); // 'orders' | 'inventory' | 'add-item'
  const [logoError, setLogoError] = useState(false);

  const [inventory, setInventory] = useState(INITIAL_INVENTORY);
  const [orders, setOrders] = useState(INITIAL_ORDERS);

  // Storefront state
  const [selectedCategory, setSelectedCategory] = useState('All');
  const [eventStart, setEventStart] = useState('2026-11-28');
  const [eventEnd, setEventEnd] = useState('2026-11-29');
  const [cart, setCart] = useState([]);
  const [clientForm, setClientForm] = useState({ name: '', eventType: 'Wedding', venue: '' });
  const [quoteSubmitted, setQuoteSubmitted] = useState(false);

  // New asset state
  const [newItem, setNewItem] = useState({
    name: '',
    category: 'Seating',
    rate: '',
    totalStock: '',
    description: '',
  });

  const rentalDays = useMemo(() => {
    const start = new Date(eventStart);
    const end = new Date(eventEnd);
    const diff = Math.ceil((end - start) / (1000 * 60 * 60 * 24)) + 1;
    return diff > 0 ? diff : 1;
  }, [eventStart, eventEnd]);

  const getAvailableQuantity = (item, startStr, endStr) => {
    const reqStart = new Date(startStr);
    const reqEnd = new Date(endStr);

    const reserved = orders
      .filter((ord) => ord.status === 'CONFIRMED' || ord.status === 'DISPATCHED')
      .filter((ord) => {
        const ordStart = new Date(ord.startDate);
        const ordEnd = new Date(ord.endDate);
        return ordStart <= reqEnd && ordEnd >= reqStart;
      })
      .reduce((sum, ord) => {
        const found = ord.items.find((i) => i.itemId === item.id);
        return sum + (found ? found.qty : 0);
      }, 0);

    return Math.max(0, item.totalStock - item.damagedStock - reserved);
  };

  const addToQuote = (item) => {
    const available = getAvailableQuantity(item, eventStart, eventEnd);
    const existing = cart.find((c) => c.item.id === item.id);
    const currentQty = existing ? existing.qty : 0;

    if (currentQty + 1 > available) {
      alert(`Only ${available} units available for these selected dates.`);
      return;
    }

    if (existing) {
      setCart(cart.map((c) => (c.item.id === item.id ? { ...c, qty: c.qty + 1 } : c)));
    } else {
      setCart([...cart, { item, qty: 1 }]);
    }
  };

  const updateCartQty = (itemId, delta) => {
    setCart((prev) =>
      prev
        .map((entry) => {
          if (entry.item.id !== itemId) return entry;
          const maxAvail = getAvailableQuantity(entry.item, eventStart, eventEnd);
          const nextQty = Math.min(maxAvail, Math.max(0, entry.qty + delta));
          return { ...entry, qty: nextQty };
        })
        .filter((entry) => entry.qty > 0)
    );
  };

  const handleQuoteSubmit = (e) => {
    e.preventDefault();
    if (!clientForm.name || cart.length === 0) return;

    const newOrder = {
      id: `BRS-ORD-${101 + orders.length}`,
      clientName: clientForm.name,
      eventType: clientForm.eventType,
      startDate: eventStart,
      endDate: eventEnd,
      venue: clientForm.venue || 'TBD',
      status: 'INQUIRY',
      items: cart.map((c) => ({ itemId: c.item.id, qty: c.qty, rate: c.item.rate })),
    };

    setOrders([newOrder, ...orders]);
    setCart([]);
    setClientForm({ name: '', eventType: 'Wedding', venue: '' });
    setQuoteSubmitted(true);
    setTimeout(() => setQuoteSubmitted(false), 4000);
  };

  const updateOrderStatus = (orderId, newStatus) => {
    setOrders(orders.map((o) => (o.id === orderId ? { ...o, status: newStatus } : o)));
  };

  const handleAddInventory = (e) => {
    e.preventDefault();
    if (!newItem.name || !newItem.rate || !newItem.totalStock) return;
    const created = {
      id: `BRS-00${inventory.length + 1}`,
      name: newItem.name,
      category: newItem.category,
      rate: parseFloat(newItem.rate),
      totalStock: parseInt(newItem.totalStock, 10),
      damagedStock: 0,
      image: '✨',
      description: newItem.description || 'Event decor & rental material from BRESONA Events.',
    };
    setInventory([...inventory, created]);
    setNewItem({ name: '', category: 'Seating', rate: '', totalStock: '', description: '' });
    setAdminTab('inventory');
  };

  const categories = ['All', ...new Set(inventory.map((i) => i.category))];
  const visibleCatalog =
    selectedCategory === 'All'
      ? inventory
      : inventory.filter((i) => i.category === selectedCategory);

  const cartSubtotal = cart.reduce((sum, c) => sum + c.item.rate * c.qty * rentalDays, 0);

  return (
    <div className="min-h-screen bg-[#FFFDF9] text-[#2B0703] font-sans">
      {/* Top Gold Accent Bar matching logo bars */}
      <div className="h-1.5 w-full bg-gradient-to-r from-[#FF5500] via-[#FFC400] to-[#FF6A00]" />

      {/* BRAND HEADER */}
      <header className="bg-white/95 backdrop-blur border-b border-orange-100 sticky top-0 z-30 shadow-sm">
        <div className="max-w-7xl mx-auto px-6 py-3.5 flex flex-wrap justify-between items-center gap-4">
          {/* Logo Display: uses /logo.png if present, with styled brand fallback */}
          <div className="flex items-center gap-3">
            {!logoError ? (
              <img
                src="/logo.png"
                alt="BRESONA EVENTS"
                onError={() => setLogoError(true)}
                className="h-12 w-auto object-contain"
              />
            ) : (
              <div className="flex flex-col">
                <div className="flex items-center gap-1">
                  <span className="h-0.5 w-6 bg-[#FFC400] rounded-full" />
                  <span className="text-[10px] font-bold tracking-[0.22em] uppercase text-[#FF6A00]">
                    Celebration & Rentals
                  </span>
                  <span className="h-0.5 w-6 bg-[#FFC400] rounded-full" />
                </div>
                <div className="flex items-center text-2xl font-black tracking-wider text-[#FF5500] leading-none mt-0.5">
                  <span>BRES</span>
                  <BresonaWinkIcon className="w-6 h-6 mx-0.5 inline-block" />
                  <span>NA</span>
                </div>
                <span className="text-[11px] font-semibold tracking-[0.38em] text-[#2B0703] mt-1">
                  EVENTS
                </span>
              </div>
            )}
          </div>

          {/* Portal Switcher */}
          <div className="flex bg-orange-50 p-1 rounded-xl border border-orange-200">
            <button
              onClick={() => setViewMode('storefront')}
              className={`px-4 py-1.5 rounded-lg text-xs font-bold transition ${
                viewMode === 'storefront'
                  ? 'bg-gradient-to-r from-[#FF5500] to-[#FF8800] text-white shadow-sm'
                  : 'text-[#2B0703]/70 hover:text-[#2B0703]'
              }`}
            >
              Client Rental Store
            </button>
            <button
              onClick={() => setViewMode('admin')}
              className={`px-4 py-1.5 rounded-lg text-xs font-bold transition ${
                viewMode === 'admin'
                  ? 'bg-gradient-to-r from-[#FF5500] to-[#FF8800] text-white shadow-sm'
                  : 'text-[#2B0703]/70 hover:text-[#2B0703]'
              }`}
            >
              Store & Warehouse Admin ({orders.length})
            </button>
          </div>
        </div>
      </header>

      {/* ================================================================= */}
      {/* VIEW 1: CLIENT STOREFRONT                                         */}
      {/* ================================================================= */}
      {viewMode === 'storefront' && (
        <div className="max-w-7xl mx-auto px-6 py-8 grid grid-cols-1 lg:grid-cols-3 gap-8">
          <div className="lg:col-span-2 space-y-6">
            {/* Hero / Date Bar */}
            <div className="bg-white border border-orange-100 rounded-2xl p-5 shadow-sm flex flex-wrap items-end justify-between gap-4">
              <div>
                <div className="inline-flex items-center gap-1.5 text-xs font-bold uppercase tracking-wider text-[#FF5500]">
                  <BresonaWinkIcon className="w-4 h-4" />
                  <span>Live Event Availability</span>
                </div>
                <p className="text-xs text-[#2B0703]/70 mt-1">
                  Pick your event dates to see real-time stock ({rentalDays} rental day
                  {rentalDays > 1 ? 's' : ''}).
                </p>
              </div>
              <div className="flex flex-wrap gap-3">
                <div>
                  <label className="block text-[11px] font-semibold text-[#2B0703]/70 mb-1">
                    Event Start
                  </label>
                  <input
                    type="date"
                    value={eventStart}
                    onChange={(e) => setEventStart(e.target.value)}
                    className="bg-[#FFFDF9] border border-orange-200 rounded-lg px-3 py-1.5 text-sm text-[#2B0703] focus:outline-none focus:border-[#FF5500]"
                  />
                </div>
                <div>
                  <label className="block text-[11px] font-semibold text-[#2B0703]/70 mb-1">
                    Event End
                  </label>
                  <input
                    type="date"
                    value={eventEnd}
                    min={eventStart}
                    onChange={(e) => setEventEnd(e.target.value)}
                    className="bg-[#FFFDF9] border border-orange-200 rounded-lg px-3 py-1.5 text-sm text-[#2B0703] focus:outline-none focus:border-[#FF5500]"
                  />
                </div>
              </div>
            </div>

            {/* Category Filter Pills */}
            <div className="flex flex-wrap gap-2">
              {categories.map((cat) => (
                <button
                  key={cat}
                  onClick={() => setSelectedCategory(cat)}
                  className={`px-4 py-1.5 rounded-full text-xs font-bold border transition ${
                    selectedCategory === cat
                      ? 'bg-[#FF5500] border-[#FF5500] text-white shadow-sm'
                      : 'bg-white border-orange-200 text-[#2B0703]/80 hover:border-[#FF8800]'
                  }`}
                >
                  {cat}
                </button>
              ))}
            </div>

            {/* Catalog Cards */}
            <div className="grid grid-cols-1 sm:grid-cols-2 gap-4">
              {visibleCatalog.map((item) => {
                const available = getAvailableQuantity(item, eventStart, eventEnd);
                return (
                  <div
                    key={item.id}
                    className="bg-white border border-orange-100 rounded-2xl p-5 flex flex-col justify-between shadow-sm hover:border-[#FF8800] transition"
                  >
                    <div>
                      <div className="flex justify-between items-start">
                        <span className="text-3xl bg-amber-50 border border-amber-200/70 w-12 h-12 rounded-xl flex items-center justify-center">
                          {item.image}
                        </span>
                        <span
                          className={`text-xs font-bold px-2.5 py-1 rounded-full ${
                            available > 10
                              ? 'bg-emerald-50 text-emerald-700 border border-emerald-200'
                              : available > 0
                              ? 'bg-amber-50 text-[#B45309] border border-amber-300'
                              : 'bg-rose-50 text-rose-700 border border-rose-200'
                          }`}
                        >
                          {available > 0 ? `${available} available` : 'Booked Out'}
                        </span>
                      </div>
                      <p className="text-[11px] font-bold uppercase tracking-wider text-[#FF6A00] mt-4">
                        {item.category} • {item.id}
                      </p>
                      <h3 className="text-base font-bold text-[#2B0703] mt-0.5">{item.name}</h3>
                      <p className="text-xs text-[#2B0703]/70 mt-1.5 leading-relaxed">
                        {item.description}
                      </p>
                    </div>

                    <div className="mt-5 pt-4 border-t border-orange-100 flex items-center justify-between">
                      <div>
                        <span className="text-lg font-extrabold text-[#2B0703]">
                          ${item.rate.toFixed(2)}
                        </span>
                        <span className="text-xs text-[#2B0703]/60"> / day</span>
                      </div>
                      <button
                        disabled={available === 0}
                        onClick={() => addToQuote(item)}
                        className="px-4 py-2 rounded-xl text-xs font-bold bg-gradient-to-r from-[#FF5500] to-[#FF8800] text-white hover:brightness-105 disabled:opacity-35 disabled:cursor-not-allowed transition shadow-sm"
                      >
                        + Add to Quote
                      </button>
                    </div>
                  </div>
                );
              })}
            </div>
          </div>

          {/* Quote Builder Sidebar */}
          <div className="bg-white border border-orange-100 rounded-2xl p-6 h-fit sticky top-24 shadow-sm">
            <div className="flex items-center justify-between border-b border-orange-100 pb-3">
              <h2 className="text-lg font-extrabold text-[#2B0703]">BRESONA Quote Cart</h2>
              <span className="text-xs font-bold px-2.5 py-0.5 rounded-full bg-amber-100 text-[#9A3412]">
                {cart.reduce((s, c) => s + c.qty, 0)} items
              </span>
            </div>

            {quoteSubmitted && (
              <div className="mt-4 p-3 rounded-xl bg-emerald-50 border border-emerald-200 text-emerald-800 text-xs font-medium">
                Inquiry received! Open the Store & Warehouse Admin tab to review and confirm.
              </div>
            )}

            {cart.length === 0 ? (
              <p className="text-sm text-[#2B0703]/50 py-10 text-center">
                Add decor or rental materials to start your BRESONA EVENTS quote.
              </p>
            ) : (
              <div className="mt-4 space-y-3 max-h-64 overflow-y-auto pr-1">
                {cart.map(({ item, qty }) => (
                  <div
                    key={item.id}
                    className="flex items-center justify-between bg-[#FFFDF9] p-3 rounded-xl border border-orange-100"
                  >
                    <div>
                      <p className="text-xs font-bold text-[#2B0703]">{item.name}</p>
                      <p className="text-[11px] text-[#2B0703]/60">
                        ${item.rate.toFixed(2)} × {qty} × {rentalDays}d
                      </p>
                    </div>
                    <div className="flex items-center gap-2">
                      <button
                        onClick={() => updateCartQty(item.id, -1)}
                        className="w-6 h-6 rounded-md bg-orange-100 text-[#2B0703] text-xs font-bold hover:bg-orange-200"
                      >
                        -
                      </button>
                      <span className="text-xs font-bold w-6 text-center">{qty}</span>
                      <button
                        onClick={() => updateCartQty(item.id, 1)}
                        className="w-6 h-6 rounded-md bg-orange-100 text-[#2B0703] text-xs font-bold hover:bg-orange-200"
                      >
                        +
                      </button>
                    </div>
                  </div>
                ))}
              </div>
            )}

            {cart.length > 0 && (
              <form
                onSubmit={handleQuoteSubmit}
                className="mt-5 pt-4 border-t border-orange-100 space-y-3"
              >
                <div className="flex justify-between text-sm font-extrabold text-[#2B0703]">
                  <span>Estimated Total ({rentalDays}d)</span>
                  <span className="text-[#FF5500]">${cartSubtotal.toFixed(2)}</span>
                </div>

                <input
                  type="text"
                  required
                  placeholder="Client / Planner Name"
                  value={clientForm.name}
                  onChange={(e) => setClientForm({ ...clientForm, name: e.target.value })}
                  className="w-full bg-[#FFFDF9] border border-orange-200 rounded-xl px-3 py-2 text-xs text-[#2B0703]"
                />
                <div className="grid grid-cols-2 gap-2">
                  <select
                    value={clientForm.eventType}
                    onChange={(e) => setClientForm({ ...clientForm, eventType: e.target.value })}
                    className="bg-[#FFFDF9] border border-orange-200 rounded-xl px-3 py-2 text-xs text-[#2B0703]"
                  >
                    <option>Wedding</option>
                    <option>Corporate Event</option>
                    <option>Birthday / Party</option>
                    <option>Exhibition</option>
                  </select>
                  <input
                    type="text"
                    placeholder="Event Venue"
                    value={clientForm.venue}
                    onChange={(e) => setClientForm({ ...clientForm, venue: e.target.value })}
                    className="bg-[#FFFDF9] border border-orange-200 rounded-xl px-3 py-2 text-xs text-[#2B0703]"
                  />
                </div>

                <button
                  type="submit"
                  className="w-full py-2.5 rounded-xl bg-gradient-to-r from-[#FF5500] to-[#FF8800] hover:brightness-105 text-white font-bold text-xs uppercase tracking-wider transition shadow-sm"
                >
                  Request Quote
                </button>
              </form>
            )}
          </div>
        </div>
      )}

      {/* ================================================================= */}
      {/* VIEW 2: STORE & WAREHOUSE ADMIN                                   */}
      {/* ================================================================= */}
      {viewMode === 'admin' && (
        <div className="max-w-7xl mx-auto px-6 py-8 space-y-6">
          <div className="flex flex-wrap gap-2 border-b border-orange-100 pb-4">
            <button
              onClick={() => setAdminTab('orders')}
              className={`px-4 py-2 rounded-xl text-xs font-bold transition ${
                adminTab === 'orders'
                  ? 'bg-[#FF5500] text-white'
                  : 'bg-white text-[#2B0703] border border-orange-200'
              }`}
            >
              Bookings & Dispatch ({orders.length})
            </button>
            <button
              onClick={() => setAdminTab('inventory')}
              className={`px-4 py-2 rounded-xl text-xs font-bold transition ${
                adminTab === 'inventory'
                  ? 'bg-[#FF5500] text-white'
                  : 'bg-white text-[#2B0703] border border-orange-200'
              }`}
            >
              Warehouse Stock ({inventory.length})
            </button>
            <button
              onClick={() => setAdminTab('add-item')}
              className={`px-4 py-2 rounded-xl text-xs font-bold transition ${
                adminTab === 'add-item'
                  ? 'bg-[#FF5500] text-white'
                  : 'bg-white text-[#2B0703] border border-orange-200'
              }`}
            >
              + Add Rental Item
            </button>
          </div>

          {adminTab === 'orders' && (
            <div className="bg-white border border-orange-100 rounded-2xl overflow-hidden shadow-sm">
              <table className="w-full text-left border-collapse">
                <thead>
                  <tr className="bg-orange-50/80 text-[#2B0703]/70 text-[11px] uppercase tracking-wider">
                    <th className="p-4">Order ID</th>
                    <th className="p-4">Client & Event</th>
                    <th className="p-4">Venue</th>
                    <th className="p-4">Dates</th>
                    <th className="p-4">Items</th>
                    <th className="p-4">Status</th>
                  </tr>
                </thead>
                <tbody className="divide-y divide-orange-100 text-sm">
                  {orders.map((ord) => (
                    <tr key={ord.id} className="hover:bg-amber-50/40">
                      <td className="p-4 font-mono text-xs text-[#FF5500] font-bold">{ord.id}</td>
                      <td className="p-4">
                        <p className="font-bold text-[#2B0703]">{ord.clientName}</p>
                        <p className="text-xs text-[#2B0703]/60">{ord.eventType}</p>
                      </td>
                      <td className="p-4 text-xs">{ord.venue}</td>
                      <td className="p-4 text-xs">
                        {ord.startDate} → {ord.endDate}
                      </td>
                      <td className="p-4 text-xs">
                        {ord.items.reduce((sum, i) => sum + i.qty, 0)} units
                      </td>
                      <td className="p-4">
                        <select
                          value={ord.status}
                          onChange={(e) => updateOrderStatus(ord.id, e.target.value)}
                          className="bg-[#FFFDF9] border border-orange-200 rounded-lg px-3 py-1.5 text-xs font-bold text-[#2B0703]"
                        >
                          <option value="INQUIRY">INQUIRY</option>
                          <option value="CONFIRMED">CONFIRMED (Locks Stock)</option>
                          <option value="DISPATCHED">DISPATCHED</option>
                          <option value="RETURNED">RETURNED</option>
                        </select>
                      </td>
                    </tr>
                  ))}
                </tbody>
              </table>
            </div>
          )}

          {adminTab === 'inventory' && (
            <div className="bg-white border border-orange-100 rounded-2xl overflow-hidden shadow-sm">
              <table className="w-full text-left border-collapse">
                <thead>
                  <tr className="bg-orange-50/80 text-[#2B0703]/70 text-[11px] uppercase tracking-wider">
                    <th className="p-4">SKU</th>
                    <th className="p-4">Item Name</th>
                    <th className="p-4">Category</th>
                    <th className="p-4">Daily Rate</th>
                    <th className="p-4">Total</th>
                    <th className="p-4">Damaged</th>
                    <th className="p-4">Rentable Pool</th>
                  </tr>
                </thead>
                <tbody className="divide-y divide-orange-100 text-sm">
                  {inventory.map((item) => (
                    <tr key={item.id} className="hover:bg-amber-50/40">
                      <td className="p-4 font-mono text-xs text-[#2B0703]/60">{item.id}</td>
                      <td className="p-4 font-bold">
                        {item.image} {item.name}
                      </td>
                      <td className="p-4 text-xs">{item.category}</td>
                      <td className="p-4 font-bold text-[#FF5500]">${item.rate.toFixed(2)}</td>
                      <td className="p-4">{item.totalStock}</td>
                      <td className="p-4 text-rose-600 font-semibold">{item.damagedStock}</td>
                      <td className="p-4 font-extrabold text-emerald-700">
                        {item.totalStock - item.damagedStock}
                      </td>
                    </tr>
                  ))}
                </tbody>
              </table>
            </div>
          )}

          {adminTab === 'add-item' && (
            <form
              onSubmit={handleAddInventory}
              className="max-w-lg bg-white border border-orange-100 rounded-2xl p-6 space-y-4 shadow-sm"
            >
              <h3 className="text-base font-extrabold text-[#2B0703]">Add New BRESONA Rental Item</h3>
              <div>
                <label className="block text-xs font-semibold text-[#2B0703]/70 mb-1">
                  Item Name
                </label>
                <input
                  type="text"
                  required
                  placeholder="e.g., Festive Neon Arch Sign"
                  value={newItem.name}
                  onChange={(e) => setNewItem({ ...newItem, name: e.target.value })}
                  className="w-full bg-[#FFFDF9] border border-orange-200 rounded-xl px-3 py-2 text-sm"
                />
              </div>
              <div className="grid grid-cols-3 gap-3">
                <div>
                  <label className="block text-xs font-semibold text-[#2B0703]/70 mb-1">
                    Category
                  </label>
                  <select
                    value={newItem.category}
                    onChange={(e) => setNewItem({ ...newItem, category: e.target.value })}
                    className="w-full bg-[#FFFDF9] border border-orange-200 rounded-xl px-3 py-2 text-sm"
                  >
                    <option>Seating</option>
                    <option>Tables</option>
                    <option>Backdrops</option>
                    <option>Lighting</option>
                    <option>Linens</option>
                    <option>Centerpieces</option>
                  </select>
                </div>
                <div>
                  <label className="block text-xs font-semibold text-[#2B0703]/70 mb-1">
                    Daily Rate ($)
                  </label>
                  <input
                    type="number"
                    step="0.01"
                    required
                    value={newItem.rate}
                    onChange={(e) => setNewItem({ ...newItem, rate: e.target.value })}
                    className="w-full bg-[#FFFDF9] border border-orange-200 rounded-xl px-3 py-2 text-sm"
                  />
                </div>
                <div>
                  <label className="block text-xs font-semibold text-[#2B0703]/70 mb-1">
                    Total Stock
                  </label>
                  <input
                    type="number"
                    required
                    value={newItem.totalStock}
                    onChange={(e) => setNewItem({ ...newItem, totalStock: e.target.value })}
                    className="w-full bg-[#FFFDF9] border border-orange-200 rounded-xl px-3 py-2 text-sm"
                  />
                </div>
              </div>
              <div>
                <label className="block text-xs font-semibold text-[#2B0703]/70 mb-1">
                  Description
                </label>
                <textarea
                  rows="2"
                  value={newItem.description}
                  onChange={(e) => setNewItem({ ...newItem, description: e.target.value })}
                  className="w-full bg-[#FFFDF9] border border-orange-200 rounded-xl px-3 py-2 text-sm"
                />
              </div>
              <button
                type="submit"
                className="px-5 py-2.5 rounded-xl bg-gradient-to-r from-[#FF5500] to-[#FF8800] text-white font-bold text-xs uppercase tracking-wider shadow-sm"
              >
                Save to Catalog
              </button>
            </form>
          )}
        </div>
      )}
    </div>
  );
}