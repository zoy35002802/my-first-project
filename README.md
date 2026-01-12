import React, { useState, useMemo, useEffect } from 'react';
import { 
  LineChart, Line, XAxis, YAxis, CartesianGrid, Tooltip, ResponsiveContainer, Legend 
} from 'recharts';
import { 
  TrendingUp, 
  Package, 
  DollarSign, 
  Settings, 
  PlusCircle, 
  History,
  PieChart,
  ShoppingCart,
  CheckCircle,
  MinusCircle,
  AlertCircle
} from 'lucide-react';

const App = () => {
  // --- 狀態定義 ---
  const [activeTab, setActiveTab] = useState('inventory');
  
  // 銀價紀錄 (單位: 兩)
  const [silverPrices, setSilverPrices] = useState([
    { date: '2023-10-01', price: 2800 },
    { date: '2023-11-01', price: 2950 },
    { date: '2023-12-01', price: 3100 },
  ]);
  const [newSilverPrice, setNewSilverPrice] = useState({ date: '', price: '' });

  // 產品庫存/進貨紀錄
  const [products, setProducts] = useState([
    { 
      id: 1, 
      name: '經典銀戒 A', 
      weight: 0.5, // 兩
      factoryFee: 200, 
      brandFee: 150, 
      purchaseDate: '2023-11-15',
      quantity: 10,
      soldCount: 3,
      price: 3500 
    }
  ]);

  // 雜項與隱形成本 (包材、贈品、試錯、開發、公關、營運)
  const [expenses, setExpenses] = useState([
    { id: 1, category: '包材', name: '首飾盒', amount: 500, quantity: 100, date: '2023-11-01' },
    { id: 2, category: '開發', name: '打樣失敗', amount: 2000, quantity: 1, date: '2023-11-10' },
  ]);

  // --- 邏輯計算 ---

  // 1. 取得特定日期的銀價 (找最接近的日期)
  const getSilverPriceOnDate = (date) => {
    const sorted = [...silverPrices].sort((a, b) => new Date(b.date) - new Date(a.date));
    const priceObj = sorted.find(p => new Date(p.date) <= new Date(date));
    return priceObj ? priceObj.price : (silverPrices[0]?.price || 0);
  };

  // 2. 隱形成本總計 (用於分攤)
  const totalHiddenCosts = useMemo(() => {
    return expenses.reduce((sum, exp) => sum + Number(exp.amount), 0);
  }, [expenses]);

  // 3. 已售出總數 (用於分攤基準)
  const totalSoldCount = useMemo(() => {
    return products.reduce((sum, p) => sum + p.soldCount, 0);
  }, [products]);

  // 4. 每件已售產品應負擔的隱形成本
  const costPerSoldItem = totalSoldCount > 0 ? totalHiddenCosts / totalSoldCount : 0;

  // --- 操作處理 ---
  
  // 產品售出增加
  const handleSellProduct = (id) => {
    setProducts(products.map(p => {
      if (p.id === id && p.soldCount < p.quantity) {
        return { ...p, soldCount: p.soldCount + 1 };
      }
      return p;
    }));
  };

  // 撤回售出 (用於修正錯誤)
  const handleUndoSell = (id) => {
    setProducts(products.map(p => {
      if (p.id === id && p.soldCount > 0) {
        return { ...p, soldCount: p.soldCount - 1 };
      }
      return p;
    }));
  };

  const handleAddProduct = (e) => {
    e.preventDefault();
    const formData = new FormData(e.target);
    const newProd = {
      id: Date.now(),
      name: formData.get('name'),
      weight: Number(formData.get('weight')),
      factoryFee: Number(formData.get('factoryFee')),
      brandFee: Number(formData.get('brandFee')),
      purchaseDate: formData.get('purchaseDate'),
      quantity: Number(formData.get('quantity')),
      soldCount: 0,
      price: Number(formData.get('price'))
    };
    setProducts([...products, newProd]);
    e.target.reset();
  };

  const handleAddExpense = (e) => {
    e.preventDefault();
    const formData = new FormData(e.target);
    const newExp = {
      id: Date.now(),
      category: formData.get('category'),
      name: formData.get('name'),
      amount: Number(formData.get('amount')),
      quantity: Number(formData.get('quantity')),
      date: formData.get('date')
    };
    setExpenses([...expenses, newExp]);
    e.target.reset();
  };

  const handleAddSilverPrice = () => {
    if (newSilverPrice.date && newSilverPrice.price) {
      setSilverPrices([...silverPrices, { ...newSilverPrice, price: Number(newSilverPrice.price) }]);
      setNewSilverPrice({ date: '', price: '' });
    }
  };

  // --- 渲染組件 ---

  const Card = ({ title, children, icon: Icon }) => (
    <div className="bg-white rounded-xl shadow-sm border border-slate-200 overflow-hidden">
      <div className="px-6 py-4 border-b border-slate-100 flex items-center gap-2 bg-slate-50">
        {Icon && <Icon className="w-5 h-5 text-indigo-600" />}
        <h3 className="font-bold text-slate-800">{title}</h3>
      </div>
      <div className="p-6">{children}</div>
    </div>
  );

  return (
    <div className="min-h-screen bg-slate-50 p-4 md:p-8 font-sans text-slate-900">
      {/* Header */}
      <header className="max-w-7xl mx-auto mb-8 flex flex-col md:flex-row md:items-center justify-between gap-4">
        <div>
          <h1 className="text-3xl font-black text-slate-900 tracking-tight">銀飾珠寶後台管理系統</h1>
          <p className="text-slate-500 font-medium">會計與庫存自動化統計工具</p>
        </div>
        <div className="flex bg-white p-1 rounded-lg border border-slate-200 shadow-sm overflow-x-auto">
          {[
            { id: 'inventory', label: '庫存與售出', icon: Package },
            { id: 'expenses', label: '隱形成本/雜項', icon: Settings },
            { id: 'silver', label: '銀價趨勢', icon: TrendingUp },
            { id: 'analysis', label: '淨利分析', icon: PieChart },
          ].map((tab) => (
            <button
              key={tab.id}
              onClick={() => setActiveTab(tab.id)}
              className={`flex items-center gap-2 px-4 py-2 rounded-md transition-all whitespace-nowrap ${
                activeTab === tab.id 
                  ? 'bg-indigo-600 text-white shadow-md' 
                  : 'text-slate-600 hover:bg-slate-100'
              }`}
            >
              <tab.icon className="w-4 h-4" />
              <span className="font-semibold">{tab.label}</span>
            </button>
          ))}
        </div>
      </header>

      <main className="max-w-7xl mx-auto space-y-6">
        
        {/* 1. 庫存與進貨管理 */}
        {activeTab === 'inventory' && (
          <div className="grid grid-cols-1 lg:grid-cols-4 gap-6">
            <div className="lg:col-span-1">
              <Card title="單次進貨輸入" icon={PlusCircle}>
                <form onSubmit={handleAddProduct} className="space-y-4">
                  <div className="space-y-1">
                    <label className="text-xs font-bold text-slate-500 uppercase">產品名稱</label>
                    <input name="name" required className="w-full border rounded-lg px-3 py-2 text-sm" placeholder="例如: 925銀項鍊" />
                  </div>
                  <div className="grid grid-cols-2 gap-4">
                    <div className="space-y-1">
                      <label className="text-xs font-bold text-slate-500 uppercase">銀重 (兩)</label>
                      <input name="weight" step="0.01" required type="number" className="w-full border rounded-lg px-3 py-2 text-sm" />
                    </div>
                    <div className="space-y-1">
                      <label className="text-xs font-bold text-slate-500 uppercase">進貨數量</label>
                      <input name="quantity" required type="number" className="w-full border rounded-lg px-3 py-2 text-sm" />
                    </div>
                  </div>
                  <div className="grid grid-cols-2 gap-4">
                    <div className="space-y-1">
                      <label className="text-xs font-bold text-slate-500 uppercase">工廠工費</label>
                      <input name="factoryFee" required type="number" className="w-full border rounded-lg px-3 py-2 text-sm" />
                    </div>
                    <div className="space-y-1">
                      <label className="text-xs font-bold text-slate-500 uppercase">品牌工費</label>
                      <input name="brandFee" required type="number" className="w-full border rounded-lg px-3 py-2 text-sm" />
                    </div>
                  </div>
                  <div className="grid grid-cols-2 gap-4">
                    <div className="space-y-1">
                      <label className="text-xs font-bold text-slate-500 uppercase">進貨日期</label>
                      <input name="purchaseDate" type="date" required className="w-full border rounded-lg px-3 py-2 text-sm" />
                    </div>
                    <div className="space-y-1">
                      <label className="text-xs font-bold text-slate-500 uppercase">銷售定價</label>
                      <input name="price" required type="number" className="w-full border rounded-lg px-3 py-2 text-sm" />
                    </div>
                  </div>
                  <button type="submit" className="w-full bg-indigo-600 text-white font-bold py-3 rounded-lg hover:bg-indigo-700 transition-colors">
                    確認入庫
                  </button>
                </form>
              </Card>
            </div>
            
            <div className="lg:col-span-3">
              <Card title="產品利潤分析與庫存售出" icon={History}>
                <div className="overflow-x-auto">
                  <table className="w-full text-left min-w-[600px]">
                    <thead>
                      <tr className="border-b text-slate-400 text-[10px] uppercase font-black">
                        <th className="px-2 py-3">產品名稱</th>
                        <th className="px-2 py-3">銀價成本</th>
                        <th className="px-2 py-3">單位總成本</th>
                        <th className="px-2 py-3">定價</th>
                        <th className="px-2 py-3">庫存狀況</th>
                        <th className="px-2 py-3">單件毛利</th>
                        <th className="px-2 py-3 text-center">售出操作</th>
                      </tr>
                    </thead>
                    <tbody className="divide-y divide-slate-100">
                      {products.map(p => {
                        const silverPrice = getSilverPriceOnDate(p.purchaseDate);
                        const silverCost = silverPrice * p.weight;
                        const unitCost = silverCost + p.factoryFee + p.brandFee;
                        const profit = p.price - unitCost;
                        const isOutOfStock = p.soldCount >= p.quantity;

                        return (
                          <tr key={p.id} className={`text-sm hover:bg-slate-50 transition-colors ${isOutOfStock ? 'opacity-60' : ''}`}>
                            <td className="px-2 py-4">
                              <div className="font-bold">{p.name}</div>
                              <div className="text-[10px] text-slate-400">{p.purchaseDate} / {p.weight}兩</div>
                            </td>
                            <td className="px-2 py-4 text-slate-500">${Math.round(silverCost).toLocaleString()}</td>
                            <td className="px-2 py-4 text-rose-600 font-medium">${Math.round(unitCost).toLocaleString()}</td>
                            <td className="px-2 py-4 text-indigo-600 font-bold">${p.price.toLocaleString()}</td>
                            <td className="px-2 py-4">
                              <div className="flex flex-col">
                                <div className="flex justify-between w-20 text-[10px] mb-1">
                                  <span>剩餘 {p.quantity - p.soldCount}</span>
                                  <span>{Math.round((p.soldCount/p.quantity)*100)}%</span>
                                </div>
                                <div className="w-20 h-1.5 bg-slate-100 rounded-full overflow-hidden">
                                  <div 
                                    className="h-full bg-indigo-500 transition-all" 
                                    style={{ width: `${(p.soldCount / p.quantity) * 100}%` }}
                                  />
                                </div>
                              </div>
                            </td>
                            <td className="px-2 py-4 font-bold text-green-600">
                              +${Math.round(profit).toLocaleString()}
                            </td>
                            <td className="px-2 py-4">
                              <div className="flex items-center justify-center gap-2">
                                <button 
                                  onClick={() => handleUndoSell(p.id)}
                                  className="p-1.5 text-slate-400 hover:text-rose-500 transition-colors"
                                  title="減少售出"
                                >
                                  <MinusCircle className="w-5 h-5" />
                                </button>
                                <button 
                                  onClick={() => handleSellProduct(p.id)}
                                  disabled={isOutOfStock}
                                  className={`flex items-center gap-1 px-3 py-1.5 rounded-lg font-bold text-xs transition-all ${
                                    isOutOfStock 
                                    ? 'bg-slate-100 text-slate-400 cursor-not-allowed' 
                                    : 'bg-green-100 text-green-700 hover:bg-green-200'
                                  }`}
                                >
                                  <ShoppingCart className="w-3 h-3" />
                                  售出
                                </button>
                              </div>
                            </td>
                          </tr>
                        );
                      })}
                    </tbody>
                  </table>
                </div>
              </Card>
            </div>
          </div>
        )}

        {/* 2. 隱形成本與雜項管理 */}
        {activeTab === 'expenses' && (
          <div className="grid grid-cols-1 lg:grid-cols-3 gap-6">
            <div className="lg:col-span-1">
              <Card title="新增支出項目" icon={PlusCircle}>
                <form onSubmit={handleAddExpense} className="space-y-4">
                  <div className="space-y-1">
                    <label className="text-xs font-bold text-slate-500 uppercase">分類</label>
                    <select name="category" className="w-full border rounded-lg px-3 py-2 text-sm">
                      <option>包材</option>
                      <option>贈品</option>
                      <option>試錯/開發</option>
                      <option>公關</option>
                      <option>營運品牌</option>
                    </select>
                  </div>
                  <div className="space-y-1">
                    <label className="text-xs font-bold text-slate-500 uppercase">項目說明</label>
                    <input name="name" required className="w-full border rounded-lg px-3 py-2 text-sm" placeholder="例如: IG廣告費" />
                  </div>
                  <div className="grid grid-cols-2 gap-4">
                    <div className="space-y-1">
                      <label className="text-xs font-bold text-slate-500 uppercase">金額</label>
                      <input name="amount" required type="number" className="w-full border rounded-lg px-3 py-2 text-sm" placeholder="$" />
                    </div>
                    <div className="space-y-1">
                      <label className="text-xs font-bold text-slate-500 uppercase">數量</label>
                      <input name="quantity" required type="number" className="w-full border rounded-lg px-3 py-2 text-sm" placeholder="1" />
                    </div>
                  </div>
                  <div className="space-y-1">
                    <label className="text-xs font-bold text-slate-500 uppercase">日期</label>
                    <input name="date" type="date" required className="w-full border rounded-lg px-3 py-2 text-sm" />
                  </div>
                  <button type="submit" className="w-full bg-slate-800 text-white font-bold py-3 rounded-lg hover:bg-slate-900 transition-colors text-sm">
                    記錄支出
                  </button>
                </form>
              </Card>
            </div>
            
            <div className="lg:col-span-2 space-y-6">
              <div className="bg-indigo-900 text-white p-6 rounded-xl shadow-lg flex flex-col md:flex-row justify-between items-center gap-4">
                <div>
                  <p className="text-indigo-200 text-xs font-bold uppercase tracking-wider">目前總隱形成本</p>
                  <p className="text-4xl font-black">${totalHiddenCosts.toLocaleString()}</p>
                </div>
                <div className="bg-indigo-800/50 p-4 rounded-lg text-center md:text-right border border-indigo-700">
                  <p className="text-indigo-200 text-[10px] font-bold uppercase tracking-wider">每件已售出負擔</p>
                  <p className="text-3xl font-bold text-indigo-300">
                    + ${Math.round(costPerSoldItem).toLocaleString()}
                  </p>
                  <p className="text-[10px] text-indigo-400 mt-1">分攤基數：{totalSoldCount} 件已售產品</p>
                </div>
              </div>
              
              <Card title="雜項與營運支出列表" icon={History}>
                <div className="overflow-x-auto">
                  <table className="w-full text-left min-w-[400px]">
                    <thead>
                      <tr className="border-b text-slate-400 text-xs uppercase font-black">
                        <th className="px-2 py-3">日期</th>
                        <th className="px-2 py-3">分類</th>
                        <th className="px-2 py-3">內容</th>
                        <th className="px-2 py-3 text-right">金額</th>
                      </tr>
                    </thead>
                    <tbody className="divide-y divide-slate-100">
                      {expenses.map(e => (
                        <tr key={e.id} className="text-sm">
                          <td className="px-2 py-4 text-slate-500 whitespace-nowrap">{e.date}</td>
                          <td className="px-2 py-4">
                            <span className="bg-slate-100 text-slate-600 px-2 py-0.5 rounded text-[10px] font-black uppercase border border-slate-200">
                              {e.category}
                            </span>
                          </td>
                          <td className="px-2 py-4 font-medium">{e.name}</td>
                          <td className="px-2 py-4 text-right font-bold">${e.amount.toLocaleString()}</td>
                        </tr>
                      ))}
                    </tbody>
                  </table>
                </div>
              </Card>
            </div>
          </div>
        )}

        {/* 3. 銀價趨勢 */}
        {activeTab === 'silver' && (
          <Card title="銀價趨勢分析 (單位: 元/兩)" icon={TrendingUp}>
            <div className="grid grid-cols-1 lg:grid-cols-4 gap-8">
              <div className="lg:col-span-1 space-y-4">
                <div className="bg-slate-50 p-4 rounded-lg border border-slate-200">
                  <h4 className="text-xs font-black text-slate-500 uppercase mb-3">輸入新數據</h4>
                  <div className="space-y-3">
                    <input 
                      type="date" 
                      className="w-full border rounded px-3 py-2 text-sm"
                      value={newSilverPrice.date}
                      onChange={e => setNewSilverPrice({...newSilverPrice, date: e.target.value})}
                    />
                    <input 
                      type="number" 
                      className="w-full border rounded px-3 py-2 text-sm"
                      placeholder="每兩價格"
                      value={newSilverPrice.price}
                      onChange={e => setNewSilverPrice({...newSilverPrice, price: e.target.value})}
                    />
                    <button 
                      onClick={handleAddSilverPrice}
                      className="w-full bg-indigo-600 text-white text-sm font-bold py-2 rounded hover:bg-indigo-700 transition-all"
                    >
                      新增紀錄
                    </button>
                  </div>
                </div>
                <div className="max-h-60 overflow-y-auto pr-2">
                  <p className="text-[10px] font-black text-slate-400 mb-2 uppercase tracking-widest">歷史紀錄</p>
                  {silverPrices.slice().reverse().map((p, i) => (
                    <div key={i} className="flex justify-between py-2 border-b text-xs last:border-0">
                      <span className="text-slate-500">{p.date}</span>
                      <span className="font-bold text-slate-700">${p.price}/兩</span>
                    </div>
                  ))}
                </div>
              </div>
              <div className="lg:col-span-3 h-80">
                <ResponsiveContainer width="100%" height="100%">
                  <LineChart data={silverPrices.sort((a,b) => new Date(a.date) - new Date(b.date))}>
                    <CartesianGrid strokeDasharray="3 3" vertical={false} stroke="#f1f5f9" />
                    <XAxis dataKey="date" fontSize={10} tick={{fill: '#94a3b8'}} axisLine={false} tickLine={false} dy={10} />
                    <YAxis fontSize={10} domain={['auto', 'auto']} tick={{fill: '#94a3b8'}} axisLine={false} tickLine={false} dx={-10} />
                    <Tooltip 
                      contentStyle={{ borderRadius: '12px', border: 'none', boxShadow: '0 10px 15px -3px rgb(0 0 0 / 0.1)', padding: '12px' }}
                      labelStyle={{ fontWeight: 'bold', marginBottom: '4px' }}
                    />
                    <Line type="monotone" dataKey="price" name="銀價" stroke="#4f46e5" strokeWidth={4} dot={{ r: 4, fill: '#4f46e5', strokeWidth: 2, stroke: '#fff' }} activeDot={{ r: 6 }} />
                  </LineChart>
                </ResponsiveContainer>
              </div>
            </div>
          </Card>
        )}

        {/* 4. 淨利分析 */}
        {activeTab === 'analysis' && (
          <div className="space-y-6">
            <div className="grid grid-cols-1 md:grid-cols-3 gap-6">
              <div className="bg-white p-6 rounded-xl border border-slate-200 shadow-sm border-l-4 border-l-indigo-500">
                <p className="text-slate-500 text-xs font-bold uppercase tracking-wider mb-1">累計銷售總額 (Revenue)</p>
                <p className="text-3xl font-black text-slate-900">
                  ${products.reduce((sum, p) => sum + (p.soldCount * p.price), 0).toLocaleString()}
                </p>
              </div>
              <div className="bg-white p-6 rounded-xl border border-slate-200 shadow-sm border-l-4 border-l-rose-500">
                <p className="text-slate-500 text-xs font-bold uppercase tracking-wider mb-1">產品直接成本 (COGS)</p>
                <p className="text-3xl font-black text-slate-900">
                  ${products.reduce((sum, p) => {
                    const silverPrice = getSilverPriceOnDate(p.purchaseDate);
                    return sum + (p.soldCount * (silverPrice * p.weight + p.factoryFee + p.brandFee));
                  }, 0).toLocaleString()}
                </p>
              </div>
              <div className="bg-white p-6 rounded-xl border border-slate-200 shadow-sm border-l-4 border-l-orange-500">
                <p className="text-slate-500 text-xs font-bold uppercase tracking-wider mb-1">已認列隱形成本 (Expenses)</p>
                <p className="text-3xl font-black text-slate-900">
                  ${Math.round(totalHiddenCosts).toLocaleString()}
                </p>
              </div>
            </div>

            <Card title="品牌經營真實淨利 (Net Profit)" icon={DollarSign}>
              <div className="flex flex-col items-center py-10 px-4">
                <div className="text-center mb-8">
                  <p className="text-slate-400 font-bold mb-2 uppercase tracking-widest text-xs">已扣除所有進貨、工費、銀價價差及雜支分攤</p>
                  <h2 className={`text-7xl font-black tracking-tighter ${
                    (products.reduce((sum, p) => {
                      const silverPrice = getSilverPriceOnDate(p.purchaseDate);
                      const unitCost = (silverPrice * p.weight + p.factoryFee + p.brandFee);
                      return sum + (p.soldCount * (p.price - unitCost));
                    }, 0) - totalHiddenCosts) >= 0 ? 'text-green-600' : 'text-rose-600'
                  }`}>
                    ${Math.round(
                      products.reduce((sum, p) => {
                        const silverPrice = getSilverPriceOnDate(p.purchaseDate);
                        const unitCost = (silverPrice * p.weight + p.factoryFee + p.brandFee);
                        return sum + (p.soldCount * (p.price - unitCost));
                      }, 0) - totalHiddenCosts
                    ).toLocaleString()}
                  </h2>
                </div>
                
                <div className="w-full max-w-2xl grid grid-cols-1 md:grid-cols-2 gap-4">
                  <div className="bg-slate-50 p-5 rounded-2xl border border-slate-200">
                    <div className="flex items-center gap-2 mb-3">
                      <Package className="w-4 h-4 text-indigo-500" />
                      <span className="text-sm font-black text-slate-700 uppercase">資產狀況</span>
                    </div>
                    <div className="space-y-2">
                      <div className="flex justify-between text-sm">
                        <span className="text-slate-500">剩餘庫存價值</span>
                        <span className="font-bold">
                          ${Math.round(products.reduce((sum, p) => {
                            const silverPrice = getSilverPriceOnDate(p.purchaseDate);
                            return sum + ((p.quantity - p.soldCount) * (silverPrice * p.weight + p.factoryFee + p.brandFee));
                          }, 0)).toLocaleString()}
                        </span>
                      </div>
                      <div className="flex justify-between text-sm">
                        <span className="text-slate-500">剩餘庫存件數</span>
                        <span className="font-bold">{products.reduce((s, p) => s + (p.quantity - p.soldCount), 0)} 件</span>
                      </div>
                    </div>
                  </div>

                  <div className="bg-slate-50 p-5 rounded-2xl border border-slate-200">
                    <div className="flex items-center gap-2 mb-3">
                      <CheckCircle className="w-4 h-4 text-green-500" />
                      <span className="text-sm font-black text-slate-700 uppercase">銷售指標</span>
                    </div>
                    <div className="space-y-2">
                      <div className="flex justify-between text-sm">
                        <span className="text-slate-500">總售出件數</span>
                        <span className="font-bold text-green-600">{totalSoldCount} 件</span>
                      </div>
                      <div className="flex justify-between text-sm">
                        <span className="text-slate-500">平均單件利潤</span>
                        <span className="font-bold">
                          ${totalSoldCount > 0 ? Math.round((products.reduce((sum, p) => {
                            const silverPrice = getSilverPriceOnDate(p.purchaseDate);
                            return sum + (p.soldCount * (p.price - (silverPrice * p.weight + p.factoryFee + p.brandFee)));
                          }, 0) - totalHiddenCosts) / totalSoldCount).toLocaleString() : 0}
                        </span>
                      </div>
                    </div>
                  </div>
                </div>

                <div className="mt-10 flex items-center gap-2 text-slate-400 bg-slate-100 px-4 py-2 rounded-full">
                  <AlertCircle className="w-4 h-4" />
                  <span className="text-[10px] font-bold">註：隱形成本的分攤會隨著「總售出件數」增加而遞減，利潤將動態成長。</span>
                </div>
              </div>
            </Card>
          </div>
        )}
      </main>
      
      {/* Footer Info */}
      <footer className="max-w-7xl mx-auto mt-12 pt-8 border-t border-slate-200 text-center text-slate-400 text-[10px] uppercase font-bold tracking-widest mb-10">
        <p>Jewelry Accounting & Inventory Management Engine</p>
        <p className="mt-1">分攤公式：Net Profit = Σ(Sold × (Price - Direct Cost)) - Total Hidden Expenses</p>
      </footer>
    </div>
  );
};

export default App;

