/*
Sigortacı F - Web Tabanlı Demo (Tek Dosya React Bileşeni)

Nasıl kullanılır:
1) Yeni bir React projesi oluşturun (ör. `npm create vite@latest demo --template react`)
2) Tailwind CSS kurun (Tailwind dokümantasyonuna göre)
3) Bu dosyayı `src/App.jsx` olarak kaydedin ve projeyi çalıştırın (`npm install` -> `npm run dev`).

Not: Bu demo gerçek e-posta/SMS gönderimi yapmaz. "Hatırlatma Gönder" butonları mock (alert) davranışı gösterir.

Özellikler gösterimi:
- Dashboard: toplam müşteri & aktif poliçe sayısı
- Müşteri listesi ve poliçe listesi
- "Bitiş tarihine 14 gün kalan" poliçeler için uyarı bandı ve filtre
- Hızlı teklif ekleme modalı (basit)
*/

import React, { useMemo, useState } from "react";

export default function App() {
  // Örnek demo verisi
  const [customers, setCustomers] = useState([
    { id: 1, name: "Ahmet Yılmaz", phone: "05001112233", email: "ahmet@example.com" },
    { id: 2, name: "Mehmet Demir", phone: "05002223344", email: "mehmet@example.com" },
    { id: 3, name: "Ayşe Kara", phone: "05003334455", email: "ayse@example.com" }
  ]);

  const [policies, setPolicies] = useState([
    { id: 101, policyNo: "TRF001", type: "Trafik", customerId: 1, start: "2025-01-01", end: "2025-08-24", premium: 3500, paymentStatus: "Ödendi" },
    { id: 102, policyNo: "KSK002", type: "Kasko", customerId: 2, start: "2025-02-01", end: "2025-08-15", premium: 7500, paymentStatus: "Beklemede" },
    { id: 103, policyNo: "KNT003", type: "Konut", customerId: 3, start: "2025-08-01", end: "2025-08-20", premium: 2500, paymentStatus: "Ödendi" }
  ]);

  const [query, setQuery] = useState("");
  const [showAdd, setShowAdd] = useState(false);

  const today = new Date();

  // kalan gün hesaplama
  const withDaysLeft = useMemo(() => {
    return policies.map(p => {
      const end = new Date(p.end + "T00:00:00");
      const diff = Math.ceil((end - today) / (1000 * 60 * 60 * 24));
      return { ...p, daysLeft: diff };
    });
  }, [policies, today]);

  const expiringSoon = withDaysLeft.filter(p => p.daysLeft <= 14 && p.daysLeft >= 0);

  // quick add policy (demo)
  function handleAddPolicy(e) {
    e.preventDefault();
    const fd = new FormData(e.target);
    const newPolicy = {
      id: Date.now(),
      policyNo: fd.get("policyNo"),
      type: fd.get("type"),
      customerId: Number(fd.get("customerId")),
      start: fd.get("start"),
      end: fd.get("end"),
      premium: Number(fd.get("premium")),
      paymentStatus: fd.get("paymentStatus")
    };
    setPolicies(prev => [newPolicy, ...prev]);
    setShowAdd(false);
  }

  function sendReminder(policy) {
    // demo: gerçek SMS/Email entegrasyonu yerine mock alert
    alert(`Hatırlatma gönderildi:\nPoliçe: ${policy.policyNo} - Müşteri ID: ${policy.customerId}`);
  }

  function formatDate(d) {
    const dt = new Date(d + "T00:00:00");
    return dt.toLocaleDateString();
  }

  const filtered = withDaysLeft.filter(p => {
    if (!query) return true;
    return p.policyNo.toLowerCase().includes(query.toLowerCase()) || p.type.toLowerCase().includes(query.toLowerCase());
  });

  return (
    <div className="min-h-screen bg-slate-50 p-6">
      <div className="max-w-6xl mx-auto">
        <header className="flex items-center justify-between mb-6">
          <h1 className="text-2xl font-semibold">Sigortacı F - Yönetim Paneli (Demo)</h1>
          <div className="text-sm text-slate-600">Gaziantep • Demo</div>
        </header>

        {/* Uyarı bandı */}
        <div className={`rounded-md p-4 mb-6 ${expiringSoon.length ? "bg-amber-50 border border-amber-200" : "bg-slate-100"}`}>
          <div className="flex items-center justify-between">
            <div>
              <strong>Yaklaşan Yenilemeler</strong>
              <div className="text-sm text-slate-600">Bitiş tarihine 14 gün veya daha az kalan poliçeler burada görünür.</div>
            </div>
            <div className="text-right">
              <div className="text-xl font-bold">{expiringSoon.length}</div>
              <div className="text-sm text-slate-500">14 gün içinde</div>
            </div>
          </div>

          {expiringSoon.length > 0 && (
            <div className="mt-4 grid gap-3 sm:grid-cols-2 lg:grid-cols-3">
              {expiringSoon.map(p => (
                <div key={p.id} className="p-3 bg-white rounded shadow-sm border">
                  <div className="flex items-center justify-between">
                    <div>
                      <div className="text-sm text-slate-500">{p.type} • {p.policyNo}</div>
                      <div className="font-medium">Müşteri ID: {p.customerId}</div>
                      <div className="text-xs text-slate-500">Bitiş: {formatDate(p.end)}</div>
                    </div>
                    <div className="text-right">
                      <div className="text-lg font-semibold">{p.daysLeft} gün</div>
                      <button onClick={() => sendReminder(p)} className="mt-2 inline-block px-3 py-1 text-sm rounded bg-blue-600 text-white">Hatırlatma Gönder</button>
                    </div>
                  </div>
                </div>
              ))}
            </div>
          )}
        </div>

        {/* Dashboard kartları */}
        <div className="grid grid-cols-1 sm:grid-cols-3 gap-4 mb-6">
          <div className="bg-white rounded p-4 shadow-sm border">
            <div className="text-sm text-slate-500">Toplam Müşteri</div>
            <div className="text-2xl font-bold">{customers.length}</div>
          </div>
          <div className="bg-white rounded p-4 shadow-sm border">
            <div className="text-sm text-slate-500">Toplam Poliçe</div>
            <div className="text-2xl font-bold">{policies.length}</div>
          </div>
          <div className="bg-white rounded p-4 shadow-sm border">
            <div className="text-sm text-slate-500">14 Gün İçinde Bitiyor</div>
            <div className="text-2xl font-bold">{expiringSoon.length}</div>
          </div>
        </div>

        {/* Arama ve işlem butonları */}
        <div className="flex items-center justify-between mb-4">
          <input value={query} onChange={e => setQuery(e.target.value)} placeholder="Poliçe No ya da Tür ile ara..." className="px-3 py-2 border rounded w-1/2" />
          <div className="flex items-center gap-2">
            <button onClick={() => setShowAdd(true)} className="px-4 py-2 bg-green-600 text-white rounded">Yeni Poliçe Ekle</button>
            <button onClick={() => alert('Demo: CSV/Excel export (gerçek uygulamada backend gerekir)')} className="px-4 py-2 border rounded">Dışa Aktar</button>
          </div>
        </div>

        {/* Poliçe listesi */}
        <div className="bg-white rounded shadow-sm border overflow-x-auto">
          <table className="min-w-full divide-y">
            <thead className="bg-slate-50">
              <tr>
                <th className="px-4 py-3 text-left text-sm text-slate-600">Poliçe No</th>
                <th className="px-4 py-3 text-left text-sm text-slate-600">Tür</th>
                <th className="px-4 py-3 text-left text-sm text-slate-600">Müşteri</th>
                <th className="px-4 py-3 text-left text-sm text-slate-600">Bitiş</th>
                <th className="px-4 py-3 text-left text-sm text-slate-600">Kalan Gün</th>
                <th className="px-4 py-3 text-left text-sm text-slate-600">Prim</th>
                <th className="px-4 py-3 text-left text-sm text-slate-600">Durum</th>
                <th className="px-4 py-3 text-right text-sm text-slate-600">İşlemler</th>
              </tr>
            </thead>
            <tbody className="bg-white divide-y">
              {filtered.map(p => {
                const cust = customers.find(c => c.id === p.customerId) || { name: 'Bilinmiyor' };
                const rowClass = p.daysLeft <= 14 && p.daysLeft >= 0 ? 'bg-amber-50' : '';
                return (
                  <tr key={p.id} className={rowClass}>
                    <td className="px-4 py-3 text-sm">{p.policyNo}</td>
                    <td className="px-4 py-3 text-sm">{p.type}</td>
                    <td className="px-4 py-3 text-sm">{cust.name}</td>
                    <td className="px-4 py-3 text-sm">{formatDate(p.end)}</td>
                    <td className="px-4 py-3 text-sm">{p.daysLeft >= 0 ? `${p.daysLeft} gün` : 'Süresi geçti'}</td>
                    <td className="px-4 py-3 text-sm">{p.premium} ₺</td>
                    <td className="px-4 py-3 text-sm">{p.paymentStatus}</td>
                    <td className="px-4 py-3 text-sm text-right">
                      <button onClick={() => sendReminder(p)} className="px-3 py-1 mr-2 border rounded">Hatırlatma</button>
                      <button onClick={() => alert('Demo: Poliçe düzenleme modalı')} className="px-3 py-1 border rounded">Düzenle</button>
                    </td>
                  </tr>
                );
              })}
            </tbody>
          </table>
        </div>

        {/* Basit müşteri kartları */}
        <section className="mt-6">
          <h2 className="text-lg font-semibold mb-3">Müşteriler</h2>
          <div className="grid grid-cols-1 sm:grid-cols-3 gap-4">
            {customers.map(c => (
              <div key={c.id} className="bg-white p-4 rounded shadow-sm border">
                <div className="font-medium">{c.name}</div>
                <div className="text-sm text-slate-500">{c.phone}</div>
                <div className="text-sm text-slate-500">{c.email}</div>
                <div className="mt-3 flex gap-2">
                  <button onClick={() => alert('Demo: Müşteri detayları')} className="px-3 py-1 border rounded">Detay</button>
                </div>
              </div>
            ))}
          </div>
        </section>

        {/* Yeni poliçe modal (basit) */}
        {showAdd && (
          <div className="fixed inset-0 bg-black/40 flex items-center justify-center">
            <div className="bg-white rounded p-6 w-full max-w-xl">
              <h3 className="text-lg font-semibold mb-3">Yeni Poliçe Ekle</h3>
              <form onSubmit={handleAddPolicy} className="grid grid-cols-1 gap-3">
                <input name="policyNo" placeholder="Poliçe No" className="px-3 py-2 border rounded" required />
                <input name="type" placeholder="Tür (Trafik/Kasko/...)" className="px-3 py-2 border rounded" required />
                <select name="customerId" className="px-3 py-2 border rounded" required>
                  <option value="">Müşteri seç</option>
                  {customers.map(c => <option key={c.id} value={c.id}>{c.name}</option>)}
                </select>
                <div className="grid grid-cols-2 gap-2">
                  <input name="start" type="date" className="px-3 py-2 border rounded" required />
                  <input name="end" type="date" className="px-3 py-2 border rounded" required />
                </div>
                <input name="premium" type="number" placeholder="Prim Tutarı" className="px-3 py-2 border rounded" required />
                <select name="paymentStatus" className="px-3 py-2 border rounded">
                  <option>Ödendi</option>
                  <option>Beklemede</option>
                </select>
                <div className="flex justify-end gap-2 mt-2">
                  <button type="button" onClick={() => setShowAdd(false)} className="px-4 py-2 border rounded">İptal</button>
                  <button type="submit" className="px-4 py-2 bg-blue-600 text-white rounded">Ekle</button>
                </div>
              </form>
            </div>
          </div>
        )}

        <footer className="mt-8 text-sm text-slate-500">Demo uygulama — gerçek kullanım için backend, veritabanı ve SMS/Email entegrasyonu gereklidir.</footer>
      </div>
    </div>
  );
}
