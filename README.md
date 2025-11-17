import React, { useState, useEffect } from 'react';
import { ArrowLeft, Download, Plus, Trash2, ChevronDown, ChevronUp } from 'lucide-react';

export default function BismillahPharmacy() {
  const [screen, setScreen] = useState('dashboard');
  const [data, setData] = useState({
    bikroy: [],
    kroy: [],
    khoroch: [],
    baki: [],
    supplier: [],
    personal: [],
    initialCash: 0
  });

  useEffect(() => {
    const saved = localStorage.getItem('bismillah-pharmacy-data');
    if (saved) {
      setData(JSON.parse(saved));
    }
  }, []);

  useEffect(() => {
    localStorage.setItem('bismillah-pharmacy-data', JSON.stringify(data));
  }, [data]);

  const menuItems = [
    { id: 'bikroy', label: 'বিক্রয়', color: 'bg-green-500', textColor: 'text-green-700' },
    { id: 'kroy', label: 'ক্রয়', color: 'bg-blue-500', textColor: 'text-blue-700' },
    { id: 'khoroch', label: 'খরচ', color: 'bg-red-500', textColor: 'text-red-700' },
    { id: 'baki', label: 'বাকি', color: 'bg-yellow-500', textColor: 'text-yellow-700' },
    { id: 'supplier', label: 'সাপ্লায়ার/পাওনাদার', color: 'bg-purple-500', textColor: 'text-purple-700' },
    { id: 'personal', label: 'ব্যক্তিগত', color: 'bg-pink-500', textColor: 'text-pink-700' },
  ];

  const downloadBackup = () => {
    const dataStr = JSON.stringify(data, null, 2);
    const dataBlob = new Blob([dataStr], { type: 'application/json' });
    const url = URL.createObjectURL(dataBlob);
    const link = document.createElement('a');
    link.href = url;
    link.download = `bismillah-pharmacy-backup-${new Date().toLocaleDateString('bn-BD')}.json`;
    link.click();
  };

  if (screen === 'dashboard') {
    return (
      <div className="min-h-screen bg-gradient-to-br from-green-50 to-blue-50 pb-20">
        <div className="bg-gradient-to-r from-green-600 to-blue-600 text-white p-6 shadow-lg">
          <div className="flex items-center justify-center mb-2">
            <div className="w-16 h-16 bg-white rounded-full flex items-center justify-center text-3xl font-bold text-green-600 shadow-lg">
              ب ف
            </div>
          </div>
          <h1 className="text-2xl font-bold text-center">বিসমিল্লাহ ফার্মেসী</h1>
          <p className="text-center text-sm opacity-90 mt-1">ম্যানেজমেন্ট সিস্টেম</p>
        </div>

        <div className="p-4">
          <div className="bg-gradient-to-r from-teal-500 to-cyan-500 text-white rounded-xl shadow-lg p-4 mb-4">
            <div className="flex items-center justify-between">
              <div className="flex items-center gap-3">
                <div className="w-12 h-12 bg-white rounded-full flex items-center justify-center">
                  <span className="text-3xl">💰</span>
                </div>
                <div>
                  <div className="text-sm opacity-90">প্রারম্ভিক ক্যাশ</div>
                  <div className="text-2xl font-bold">৳{(data.initialCash || 0).toLocaleString('bn-BD')}</div>
                </div>
              </div>
              <button
                onClick={() => setScreen('initialCash')}
                className="bg-white text-teal-600 px-4 py-2 rounded-lg font-semibold hover:bg-opacity-90 transition-all"
              >
                পরিবর্তন
              </button>
            </div>
          </div>

          <div className="grid grid-cols-2 gap-4 max-w-2xl mx-auto">
            {menuItems.map((item) => (
              <button
                key={item.id}
                onClick={() => setScreen(item.id)}
                className={`${item.color} text-white p-6 rounded-xl shadow-lg hover:shadow-xl transform hover:scale-105 transition-all duration-200 font-bold text-lg`}
              >
                {item.label}
              </button>
            ))}
          </div>
        </div>

        <div className="fixed bottom-6 right-6">
          <button
            onClick={downloadBackup}
            className="bg-gradient-to-r from-orange-500 to-red-500 text-white p-4 rounded-full shadow-2xl hover:shadow-3xl transform hover:scale-110 transition-all"
          >
            <Download size={24} />
          </button>
        </div>
      </div>
    );
  }

  if (screen === 'initialCash') {
    return <InitialCashScreen data={data} setData={setData} setScreen={setScreen} />;
  }

  if (screen === 'baki') {
    return <BakiScreen data={data} setData={setData} setScreen={setScreen} />;
  }

  if (screen === 'supplier') {
    return <SupplierScreen data={data} setData={setData} setScreen={setScreen} />;
  }

  return <RecordScreen screen={screen} data={data} setData={setData} setScreen={setScreen} menuItems={menuItems} />;
}

function RecordScreen({ screen, data, setData, setScreen, menuItems }) {
  const [product, setProduct] = useState('');
  const [amount, setAmount] = useState('');
  const [suggestions, setSuggestions] = useState([]);
  const [showSuggestions, setShowSuggestions] = useState(false);
  const [showRecords, setShowRecords] = useState(false);

  const currentMenu = menuItems.find(m => m.id === screen);
  const records = data[screen] || [];

  const allProducts = [...new Set(records.map(r => r.product))];

  const handleProductChange = (value) => {
    setProduct(value);
    if (value) {
      const filtered = allProducts.filter(p => p.toLowerCase().includes(value.toLowerCase()));
      setSuggestions(filtered);
      setShowSuggestions(filtered.length > 0);
    } else {
      setShowSuggestions(false);
    }
  };

  const addRecord = () => {
    if (product && amount) {
      const newRecord = {
        id: Date.now(),
        product,
        amount: parseFloat(amount),
        date: new Date().toLocaleDateString('bn-BD')
      };
      setData({ ...data, [screen]: [...records, newRecord] });
      setProduct('');
      setAmount('');
      setShowSuggestions(false);
    }
  };

  const deleteRecord = (id) => {
    setData({ ...data, [screen]: records.filter(r => r.id !== id) });
  };

  const total = records.reduce((sum, r) => sum + r.amount, 0);

  return (
    <div className="min-h-screen bg-gray-50 pb-64">
      <div className={`${currentMenu.color} text-white p-4 shadow-lg flex items-center sticky top-0 z-20`}>
        <button onClick={() => setScreen('dashboard')} className="mr-3">
          <ArrowLeft size={24} />
        </button>
        <h1 className="text-xl font-bold">{currentMenu.label} রেকর্ড</h1>
      </div>

      <div className="p-4">
        <button
          onClick={() => setShowRecords(!showRecords)}
          className={`w-full ${currentMenu.color} text-white p-4 rounded-xl shadow-lg mb-4 flex items-center justify-between font-bold text-lg`}
        >
          <span>মোট যোগফল: ৳{total.toLocaleString('bn-BD')}</span>
          {showRecords ? <ChevronUp size={24} /> : <ChevronDown size={24} />}
        </button>

        {showRecords && (
          <div className="bg-white rounded-lg shadow-md overflow-hidden mb-4 max-h-96 overflow-y-auto">
            <div className="divide-y">
              {records.map((record) => (
                <div key={record.id} className="p-4 flex justify-between items-center hover:bg-gray-50">
                  <div className="flex-1">
                    <div className="font-semibold text-lg">{record.product}</div>
                    <div className="text-sm text-gray-500">{record.date}</div>
                  </div>
                  <div className="flex items-center gap-3">
                    <div className={`font-bold text-lg ${currentMenu.textColor}`}>
                      ৳{record.amount.toLocaleString('bn-BD')}
                    </div>
                    <button
                      onClick={() => deleteRecord(record.id)}
                      className="text-red-500 hover:bg-red-50 p-2 rounded"
                    >
                      <Trash2 size={20} />
                    </button>
                  </div>
                </div>
              ))}
              {records.length === 0 && (
                <div className="p-8 text-center text-gray-400">কোনো রেকর্ড নেই</div>
              )}
            </div>
          </div>
        )}
      </div>

      <div className="fixed bottom-0 left-0 right-0 bg-white shadow-2xl border-t-4 border-gray-200 p-4 z-30">
        <div className="max-w-2xl mx-auto">
          <div className="mb-3 relative">
            <label className="block text-gray-700 font-semibold mb-2">পণ্য / বিবরণ</label>
            {showSuggestions && suggestions.length > 0 && (
              <div className="absolute bottom-full left-0 right-0 bg-white border-2 border-gray-300 rounded-lg mb-2 max-h-40 overflow-y-auto shadow-lg z-50">
                {suggestions.map((sug, idx) => (
                  <div
                    key={idx}
                    onClick={() => {
                      setProduct(sug);
                      setShowSuggestions(false);
                    }}
                    className="p-3 hover:bg-blue-100 cursor-pointer border-b"
                  >
                    {sug}
                  </div>
                ))}
              </div>
            )}
            <input
              type="text"
              value={product}
              onChange={(e) => handleProductChange(e.target.value)}
              onFocus={() => allProducts.length > 0 && setShowSuggestions(true)}
              className="w-full p-3 border-2 border-gray-300 rounded-lg focus:border-blue-500 focus:outline-none text-lg"
              placeholder="পণ্যের নাম লিখুন"
            />
          </div>

          <div className="mb-3">
            <label className="block text-gray-700 font-semibold mb-2">টাকা</label>
            <input
              type="number"
              value={amount}
              onChange={(e) => setAmount(e.target.value)}
              className="w-full p-3 border-2 border-gray-300 rounded-lg focus:border-blue-500 focus:outline-none text-lg"
              placeholder="টাকার পরিমাণ"
            />
          </div>

          <button
            onClick={addRecord}
            className={`w-full ${currentMenu.color} text-white p-4 rounded-lg font-bold text-lg flex items-center justify-center gap-2 shadow-lg hover:shadow-xl transition-all`}
          >
            <Plus size={24} />
            যোগ করুন
          </button>
        </div>
      </div>
    </div>
  );
}

function InitialCashScreen({ data, setData, setScreen }) {
  const [cash, setCash] = useState(data.initialCash || 0);

  const saveCash = () => {
    setData({ ...data, initialCash: parseFloat(cash) || 0 });
    alert('প্রারম্ভিক ক্যাশ সংরক্ষিত হয়েছে!');
  };

  return (
    <div className="min-h-screen bg-gray-50">
      <div className="bg-gradient-to-r from-teal-500 to-cyan-500 text-white p-4 shadow-lg flex items-center">
        <button onClick={() => setScreen('dashboard')} className="mr-3">
          <ArrowLeft size={24} />
        </button>
        <h1 className="text-xl font-bold">প্রারম্ভিক ক্যাশ</h1>
      </div>

      <div className="p-6 max-w-md mx-auto mt-8">
        <div className="bg-white rounded-lg shadow-lg p-6">
          <label className="block text-gray-700 font-semibold mb-3 text-lg">প্রারম্ভিক টাকার পরিমাণ</label>
          <input
            type="number"
            value={cash}
            onChange={(e) => setCash(e.target.value)}
            className="w-full p-4 border-2 border-gray-300 rounded-lg focus:border-teal-500 focus:outline-none text-xl mb-4"
            placeholder="টাকা লিখুন"
          />
          <button
            onClick={saveCash}
            className="w-full bg-gradient-to-r from-teal-500 to-cyan-500 text-white p-4 rounded-lg font-bold text-lg shadow-lg hover:shadow-xl transition-all"
          >
            সংরক্ষণ করুন
          </button>
          
          <div className="mt-6 p-4 bg-teal-50 rounded-lg text-center">
            <div className="text-gray-600 mb-1">বর্তমান ক্যাশ</div>
            <div className="text-3xl font-bold text-teal-600">৳{(data.initialCash || 0).toLocaleString('bn-BD')}</div>
          </div>
        </div>
      </div>
    </div>
  );
}

function BakiScreen({ data, setData, setScreen }) {
  const [name, setName] = useState('');
  const [mobile, setMobile] = useState('');
  const [baki, setBaki] = useState('');
  const [joma, setJoma] = useState('');
  const [nameSuggestions, setNameSuggestions] = useState([]);
  const [showNameSuggestions, setShowNameSuggestions] = useState(false);
  const [showRecords, setShowRecords] = useState(false);

  const records = data.baki || [];
  const allNames = [...new Set(records.map(r => r.name))];

  const handleNameChange = (value) => {
    setName(value);
    if (value) {
      const filtered = allNames.filter(n => n.toLowerCase().includes(value.toLowerCase()));
      setNameSuggestions(filtered);
      setShowNameSuggestions(filtered.length > 0);
    } else {
      setShowNameSuggestions(false);
    }
  };

  const addRecord = () => {
    if (name) {
      const newRecord = {
        id: Date.now(),
        name,
        mobile,
        baki: parseFloat(baki) || 0,
        joma: parseFloat(joma) || 0,
        date: new Date().toLocaleDateString('bn-BD')
      };
      setData({ ...data, baki: [...records, newRecord] });
      setName('');
      setMobile('');
      setBaki('');
      setJoma('');
      setShowNameSuggestions(false);
    }
  };

  const deleteRecord = (id) => {
    setData({ ...data, baki: records.filter(r => r.id !== id) });
  };

  const totalBaki = records.reduce((sum, r) => sum + r.baki, 0);
  const totalJoma = records.reduce((sum, r) => sum + r.joma, 0);

  return (
    <div className="min-h-screen bg-gray-50 pb-96">
      <div className="bg-gradient-to-r from-yellow-500 to-orange-500 text-white p-4 shadow-lg flex items-center sticky top-0 z-20">
        <button onClick={() => setScreen('dashboard')} className="mr-3">
          <ArrowLeft size={24} />
        </button>
        <h1 className="text-xl font-bold">বাকি রেকর্ড</h1>
      </div>

      <div className="p-4">
        <button
          onClick={() => setShowRecords(!showRecords)}
          className="w-full bg-gradient-to-r from-yellow-500 to-orange-500 text-white p-4 rounded-xl shadow-lg mb-4 font-bold"
        >
          <div className="flex items-center justify-between mb-2">
            <span>সর্বমোট হিসাব</span>
            {showRecords ? <ChevronUp size={24} /> : <ChevronDown size={24} />}
          </div>
          <div className="flex justify-around text-sm">
            <div>বাকি: ৳{totalBaki.toLocaleString('bn-BD')}</div>
            <div>জমা: ৳{totalJoma.toLocaleString('bn-BD')}</div>
          </div>
        </button>

        {showRecords && (
          <div className="bg-white rounded-lg shadow-md overflow-hidden mb-4 max-h-96 overflow-y-auto">
            {records.map((record) => (
              <div key={record.id} className="p-4 border-b hover:bg-yellow-50">
                <div className="flex justify-between items-start mb-2">
                  <div>
                    <div className="font-bold text-lg">{record.name}</div>
                    <div className="text-sm text-gray-600">{record.mobile}</div>
                    <div className="text-xs text-gray-400">{record.date}</div>
                  </div>
                  <button
                    onClick={() => deleteRecord(record.id)}
                    className="text-red-500 hover:bg-red-50 p-2 rounded"
                  >
                    <Trash2 size={20} />
                  </button>
                </div>
                <div className="grid grid-cols-2 gap-2 text-sm">
                  <div className="bg-red-100 p-2 rounded">
                    <div className="text-gray-600">বাকি</div>
                    <div className="font-bold text-red-600">৳{record.baki.toLocaleString('bn-BD')}</div>
                  </div>
                  <div className="bg-green-100 p-2 rounded">
                    <div className="text-gray-600">জমা</div>
                    <div className="font-bold text-green-600">৳{record.joma.toLocaleString('bn-BD')}</div>
                  </div>
                </div>
              </div>
            ))}
            {records.length === 0 && (
              <div className="p-8 text-center text-gray-400">কোনো রেকর্ড নেই</div>
            )}
          </div>
        )}
      </div>

      <div className="fixed bottom-0 left-0 right-0 bg-white shadow-2xl border-t-4 border-gray-200 p-4 z-30">
        <div className="max-w-2xl mx-auto">
          <div className="mb-3 relative">
            <label className="block text-gray-700 font-semibold mb-2">নাম</label>
            {showNameSuggestions && nameSuggestions.length > 0 && (
              <div className="absolute bottom-full left-0 right-0 bg-white border-2 border-gray-300 rounded-lg mb-2 max-h-40 overflow-y-auto shadow-lg z-50">
                {nameSuggestions.map((sug, idx) => (
                  <div
                    key={idx}
                    onClick={() => {
                      setName(sug);
                      setShowNameSuggestions(false);
                    }}
                    className="p-3 hover:bg-yellow-100 cursor-pointer border-b"
                  >
                    {sug}
                  </div>
                ))}
              </div>
            )}
            <input
              type="text"
              value={name}
              onChange={(e) => handleNameChange(e.target.value)}
              onFocus={() => allNames.length > 0 && setShowNameSuggestions(true)}
              className="w-full p-3 border-2 border-gray-300 rounded-lg focus:border-yellow-500 focus:outline-none"
              placeholder="নাম লিখুন"
            />
          </div>

          <div className="grid grid-cols-2 gap-3 mb-3">
            <div>
              <label className="block text-gray-700 font-semibold mb-2">মোবাইল নং</label>
              <input
                type="tel"
                value={mobile}
                onChange={(e) => setMobile(e.target.value)}
                className="w-full p-3 border-2 border-gray-300 rounded-lg focus:border-yellow-500 focus:outline-none"
                placeholder="মোবাইল"
              />
            </div>
            <div>
              <label className="block text-gray-700 font-semibold mb-2">বাকি</label>
              <input
                type="number"
                value={baki}
                onChange={(e) => setBaki(e.target.value)}
                className="w-full p-3 border-2 border-gray-300 rounded-lg focus:border-yellow-500 focus:outline-none"
                placeholder="বাকি"
              />
            </div>
          </div>

          <div className="mb-3">
            <label className="block text-gray-700 font-semibold mb-2">জমা</label>
            <input
              type="number"
              value={joma}
              onChange={(e) => setJoma(e.target.value)}
              className="w-full p-3 border-2 border-gray-300 rounded-lg focus:border-yellow-500 focus:outline-none"
              placeholder="জমার পরিমাণ"
            />
          </div>

          <button
            onClick={addRecord}
            className="w-full bg-gradient-to-r from-yellow-500 to-orange-500 text-white p-4 rounded-lg font-bold text-lg flex items-center justify-center gap-2 shadow-lg"
          >
            <Plus size={24} />
            যোগ করুন
          </button>
        </div>
      </div>
    </div>
  );
}

function SupplierScreen({ data, setData, setScreen }) {
  const [name, setName] = useState('');
  const [mobile, setMobile] = useState('');
  const [total, setTotal] = useState('');
  const [joma, setJoma] = useState('');
  const [nameSuggestions, setNameSuggestions] = useState([]);
  const [showNameSuggestions, setShowNameSuggestions] = useState(false);
  const [showRecords, setShowRecords] = useState(false);

  const records = data.supplier || [];
  const allNames = [...new Set(records.map(r => r.name))];

  const handleNameChange = (value) => {
    setName(value);
    if (value) {
      const filtered = allNames.filter(n => n.toLowerCase().includes(value.toLowerCase()));
      setNameSuggestions(filtered);
      setShowNameSuggestions(filtered.length > 0);
    } else {
      setShowNameSuggestions(false);
    }
  };

  const addRecord = () => {
    if (name) {
      const totalAmount = parseFloat(total) || 0;
      const jomaAmount = parseFloat(joma) || 0;
      const newRecord = {
        id: Date.now(),
        name,
        mobile,
        total: totalAmount,
        joma: jomaAmount,
        remaining: totalAmount - jomaAmount,
        date: new Date().toLocaleDateString('bn-BD')
      };
      setData({ ...data, supplier: [...records, newRecord] });
      setName('');
      setMobile('');
      setTotal('');
      setJoma('');
      setShowNameSuggestions(false);
    }
  };

  const deleteRecord = (id) => {
    setData({ ...data, supplier: records.filter(r => r.id !== id) });
  };

  const totalAmount = records.reduce((sum, r) => sum + r.total, 0);
  const totalJoma = records.reduce((sum, r) => sum + r.joma, 0);
  const totalRemaining = records.reduce((sum, r) => sum + r.remaining, 0);

  return (
    <div className="min-h-screen bg-gray-50 pb-96">
      <div className="bg-gradient-to-r from-purple-500 to-indigo-500 text-white p-4 shadow-lg flex items-center sticky top-0 z-20">
        <button onClick={() => setScreen('dashboard')} className="mr-3">
          <ArrowLeft size={24} />
        </button>
        <h1 className="text-xl font-bold">সাপ্লায়ার/পাওনাদার</h1>
      </div>

      <div className="p-4">
        <button
          onClick={() => setShowRecords(!showRecords)}
          className="w-full bg-gradient-to-r from-purple-500 to-indigo-500 text-white p-4 rounded-xl shadow-lg mb-4 font-bold"
        >
          <div className="flex items-center justify-between mb-2">
            <span>সর্বমোট হিসাব</span>
            {showRecords ? <ChevronUp size={24} /> : <ChevronDown size={24} />}
          </div>
          <div className="grid grid-cols-3 gap-2 text-sm">
            <div>মোট: ৳{totalAmount.toLocaleString('bn-BD')}</div>
            <div>জমা: ৳{totalJoma.toLocaleString('bn-BD')}</div>
            <div>বাকি: ৳{totalRemaining.toLocaleString('bn-BD')}</div>
          </div>
        </button>

        {showRecords && (
          <div className="bg-white rounded-lg shadow-md overflow-hidden mb-4 max-h-96 overflow-y-auto">
            {records.map((record) => (
              <div key={record.id} className="p-4 border-b hover:bg-purple-50">
                <div className="flex justify-between items-start mb-2">
                  <div>
                    <div className="font-bold text-lg">{record.name}</div>
                    <div className="text-sm text-gray-600">{record.mobile}</div>
                    <div className="text-xs text-gray-400">{record.date}</div>
                  </div>
                  <button
                    onClick={() => deleteRecord(record.id)}
                    className="text-red-500 hover:bg-red-50 p-2 rounded"
                  >
                    <Trash2 size={20} />
                  </button>
                </div>
                <div className="grid grid-cols-3 gap-2 text-sm">
                  <div className="bg-gray-100 p-2 rounded">
                    <div className="text-gray-600 text-xs">মোট টাকা</div>
                    <div className="font-bold text-gray-700">৳{record.total.toLocaleString('bn-BD')}</div>
                  </div>
                  <div className="bg-green-100 p-2 rounded">
                    <div className="text-gray-600 -xs">জমা</div>
                    <div className="font-bold text-green-600">৳{record.joma.toLocaleString('bn-BD')}</div>
                  </div>
                  <div className="bg-red-100 p-2 rounded">
                    <div className="text-gray-600 text-xs">অবশিষ্ট</div>
                    <div className="font-bold text-red-600">৳{record.remaining.
