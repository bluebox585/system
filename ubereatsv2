import React, { useState, useEffect, useMemo, useRef } from 'react';
import { Clipboard, ShoppingBag, Calculator, Copy, Check, ChevronDown, ChevronRight, Download, Image as ImageIcon, Edit2, Save } from 'lucide-react';

// --- Mock Data ---
const EXAMPLE_TEXT = `莞婷 (您本人)
3 份餐點


1
南洋沙嗲牛肉幫米短法
$140.00
加辣選擇
微辣

1
塑膠袋
$2.00

1
椰香奶酥烤片
$55.00
群組中的其他人
11人中有 11人已新增餐點



燕鳳
2 份餐點


1
雪藏生乳紅茶
$75.00

1
蒜香雪花牛肉幫米短法
$150.00
加辣選擇
微辣

移除參加者


Gene
2 份餐點


1
舒肥低脂嫩雞幫米短法
$135.00
選擇
對切
加辣選擇
小辣
備註：不加香菜

1
熟成紅茶
$35.00

移除參加者


彥文
準備好了
 • 
2 份餐點


1
熟成紅茶
$35.00

1
南洋沙嗲牛肉幫米短法
$140.00
加辣選擇
微辣

移除參加者


墩
1 份餐點


1
蒜香雪花牛肉幫米短法
$150.00
加辣選擇
微辣

移除參加者


JL
2 份餐點


1
舒肥低脂嫩雞幫米短法
$135.00

1
熟成紅茶
$35.00

移除參加者


智鈺
準備好了
 • 
2 份餐點


1
低脂嫩雞米線
$150.00
加辣選擇
中辣

1
野菜香茅豬肉幫米短法
$135.00
選擇
對切
加辣選擇
微辣

移除參加者


昆穎
1 份餐點


1
舒肥低脂嫩雞幫米短法
$135.00
選擇
對切
加辣選擇
微辣

移除參加者


丁
1 份餐點


1
蒜香雪花牛肉幫米短法
$150.00
選擇
對切
加辣選擇
微辣

移除參加者


Sing
1 份餐點


1
低脂嫩雞米線
$150.00

移除參加者


Liu
4 份餐點


1
蒜香雪花牛肉幫米短法
$150.00
加辣選擇
微辣

3
泰式奶茶
$210.00

移除參加者


王
準備好了
 • 
5 份餐點


1
南洋沙嗲牛肉幫米短法
$140.00
加辣選擇
小辣

1
生乳奶油卡士達
$90.00

1
南洋沙嗲牛肉幫米短法
$140.00
加辣選擇
微辣

1
熟成紅茶
$35.00

1
野菜香茅豬肉幫米短法
$135.00`;

const App = () => {
  const [inputText, setInputText] = useState('');
  const [discount, setDiscount] = useState(50); // 修改預設折扣為 50%
  const [shopName, setShopName] = useState(''); // 新增店家名稱狀態
  const [parsedData, setParsedData] = useState([]);
  const [copyFeedback, setCopyFeedback] = useState('');
  const [isHtml2CanvasLoaded, setIsHtml2CanvasLoaded] = useState(false);
  const [isEditMode, setIsEditMode] = useState(false); // 新增編輯模式狀態
  
  const resultRef = useRef(null);
  const summaryRef = useRef(null); // 用於截圖的 Ref

  // 動態載入 html2canvas
  useEffect(() => {
    const script = document.createElement('script');
    script.src = "https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js";
    script.onload = () => setIsHtml2CanvasLoaded(true);
    document.body.appendChild(script);
    return () => {
        if(document.body.contains(script)) {
            document.body.removeChild(script);
        }
    }
  }, []);

  // --- 解析器 ---
  const parseText = () => {
    if (!inputText.trim()) {
      alert('請先貼上文字！');
      return;
    }

    const lines = inputText.split('\n').map(l => l.trim());
    const peopleBlocks = [];

    const anchorRegex = /^\d+\s*份餐點/; 
    const statusSkipRegex = /^(準備好了|•|移除參加者)$/;

    lines.forEach((line, idx) => {
        if (anchorRegex.test(line)) {
            let name = "未命名";
            let nameLineIdx = -1;

            for (let i = idx - 1; i >= 0; i--) {
                const l = lines[i];
                if (!l) continue;
                if (statusSkipRegex.test(l)) continue;
                
                name = l;
                nameLineIdx = i;
                break;
            }

            // 1. 修正名字：移除 "(您本人)"
            name = name.replace(/\s*\(您本人\)/, '').trim();

            if (peopleBlocks.length > 0) {
                peopleBlocks[peopleBlocks.length - 1].endLine = nameLineIdx;
            }

            peopleBlocks.push({
                name: name,
                startLine: idx + 1,
                endLine: lines.length
            });
        }
    });

    if (peopleBlocks.length === 0) {
        alert("找不到人員資料，請確認格式是否包含 'X 份餐點'");
        return;
    }

    const parsedPeople = peopleBlocks.map(block => {
        const blockLines = lines.slice(block.startLine, block.endLine);
        const items = parseBlockItems(blockLines);
        return { name: block.name, items };
    }).filter(p => p.items.length > 0);

    setParsedData(parsedPeople);
    setIsEditMode(false); // 解析後預設關閉編輯模式
    
    setTimeout(() => {
        resultRef.current?.scrollIntoView({ behavior: 'smooth' });
    }, 100);
  };

  const parseBlockItems = (lines) => {
    const items = [];
    let currentItem = null;
    let pendingPrefix = null; // 用來暫存 "選擇" 這種前綴

    const ignoreRegex = /(^移除參加者$|^群組中的其他人$|人中有.*人已新增餐點|^準備好了$|^•$)/;
    const qtyRegex = /^\d+$/; 
    const priceRegex = /^[$\uff04]?(\d+(\.\d+)?)$/;

    const pushCurrentItem = () => {
        if (currentItem && currentItem.name) {
            items.push(currentItem);
        }
        pendingPrefix = null; // 換新項目時，清空暫存前綴
    };

    for (let line of lines) {
        if (!line) continue;
        if (ignoreRegex.test(line)) continue;

        if (qtyRegex.test(line) && line.length < 3) {
            pushCurrentItem();
            currentItem = {
                qty: parseInt(line),
                name: '',
                price: 0,
                options: []
            };
            continue;
        }

        const priceMatch = line.match(priceRegex);
        if (priceMatch) {
            if (currentItem) {
                currentItem.price = parseFloat(priceMatch[1]);
            }
            continue;
        }

        if (currentItem) {
            if (!currentItem.name) {
                currentItem.name = line;
            } else {
                if (line !== '0') {
                    // --- 處理選項顯示邏輯 ---
                    
                    // 1. 隱藏 "加辣選擇"
                    if (line === '加辣選擇') {
                        continue; 
                    }

                    // 2. 合併 "選擇" 與下一行
                    if (line === '選擇') {
                        pendingPrefix = line;
                        continue;
                    }

                    if (pendingPrefix) {
                        currentItem.options.push(`${pendingPrefix}-${line}`);
                        pendingPrefix = null;
                    } else {
                        currentItem.options.push(line);
                    }
                }
            }
        }
    }
    pushCurrentItem();
    return items;
  };

  // --- 更新數據功能 (編輯數量或單價) ---
  const handleUpdateItem = (personIndex, itemIndex, field, value) => {
    const newData = [...parsedData];
    const person = newData[personIndex];
    const item = person.items[itemIndex];

    if (field === 'price') {
        item.price = parseFloat(value) || 0;
    } else if (field === 'qty') {
        item.qty = parseInt(value) || 0;
    }

    setParsedData(newData);
  };

  // --- 計算 ---
  const calculatedData = useMemo(() => {
    return parsedData.map((person, personIndex) => {
      const totalOriginal = person.items.reduce((sum, item) => sum + (item.price * item.qty), 0); // 這裡修正為 單價*數量
      const totalDiscounted = Math.round(totalOriginal * (discount / 100));
      return {
        ...person,
        originalIndex: personIndex, // 保留原始索引以便編輯時查找
        totalOriginal,
        totalDiscounted
      };
    });
  }, [parsedData, discount]);

  // --- 資料處理：依餐點名稱分組 (Aggregated by Item Name) ---
  const aggregatedGroups = useMemo(() => {
    const itemsMap = {}; // Key: Item Name

    parsedData.forEach(person => {
      person.items.forEach(item => {
        const name = item.name;
        
        if (!itemsMap[name]) {
          itemsMap[name] = {
            name: name,
            totalCount: 0,
            totalPrice: 0,
            variants: {} // Key: OptionString (用來區分不同的備註組合)
          };
        }
        
        const group = itemsMap[name];
        group.totalCount += item.qty;
        group.totalPrice += item.price * item.qty;

        // 使用備註當作 variant key
        const optionStr = item.options.join(' '); 
        
        if (!group.variants[optionStr]) {
            group.variants[optionStr] = {
                options: item.options,
                count: 0,
                buyers: []
            };
        }
        
        const variant = group.variants[optionStr];
        variant.count += item.qty;
        
        // Add buyer
        const existingBuyer = variant.buyers.find(b => b.name === person.name);
        if (existingBuyer) {
            existingBuyer.count += item.qty;
        } else {
            variant.buyers.push({ name: person.name, count: item.qty });
        }
      });
    });
    
    // Convert to array and inner variants to array
    return Object.values(itemsMap).map(item => ({
        ...item,
        variants: Object.values(item.variants)
    }));
  }, [parsedData]);

  // --- Actions ---
  const copyToClipboard = (text, label) => {
    const textArea = document.createElement("textarea");
    textArea.value = text;
    document.body.appendChild(textArea);
    textArea.select();
    try {
      document.execCommand('copy');
      setCopyFeedback(label);
      setTimeout(() => setCopyFeedback(''), 2000);
    } catch (err) {
      console.error('Unable to copy', err);
    }
    document.body.removeChild(textArea);
  };

  const handleExportImage = async () => {
    if (!window.html2canvas || !summaryRef.current) return;
    
    try {
        const canvas = await window.html2canvas(summaryRef.current, {
            backgroundColor: "#ffffff",
            scale: 2, // 提高解析度
            useCORS: true
        });
        
        const link = document.createElement('a');
        const timestamp = new Date().toISOString().slice(0, 10);
        link.download = `團購明細_${timestamp}.png`;
        link.href = canvas.toDataURL('image/png');
        link.click();
        
        setCopyFeedback('imageExport');
        setTimeout(() => setCopyFeedback(''), 2000);
    } catch (error) {
        console.error("截圖失敗:", error);
        alert("截圖失敗，請稍後再試");
    }
  };

  const generateOrderListText = () => {
    let text = `訂餐統計清單\n----------------\n`;
    aggregatedGroups.forEach(group => {
      text += `【${group.name}】 總數: ${group.totalCount}\n`;
      group.variants.forEach(variant => {
         const optionsStr = variant.options.length > 0 ? variant.options.join('、') : '原味/無備註';
         const buyersStr = variant.buyers.map(b => b.name + (b.count > 1 ? 'x'+b.count : '')).join('、');
         text += `  - ${optionsStr} x${variant.count} (${buyersStr})\n`;
      });
      text += '\n';
    });
    return text;
  };

  // --- UI ---

  return (
    <div className="min-h-screen bg-gray-50 font-sans text-gray-800 pb-20">
      
      {/* Header */}
      <header className="bg-white border-b border-gray-200 py-6 mb-6 shadow-sm sticky top-0 z-10">
        <div className="max-w-4xl mx-auto px-4 text-center">
          <h1 className="text-3xl font-bold text-green-600 mb-2 flex items-center justify-center gap-2">
            <ShoppingBag /> UberEats 團購車整理神器
          </h1>
          <p className="text-gray-500">支援新版格式！貼上複製文字，一鍵產出清爽表格</p>
        </div>
      </header>

      <main className="max-w-4xl mx-auto px-4 space-y-8">
        
        {/* Step 1 */}
        <section className="bg-white rounded-xl shadow-lg p-6 md:p-8 animate-fade-in border border-gray-100">
          <div className="flex justify-between items-center mb-4">
            <h2 className="text-2xl font-bold text-gray-800 flex items-center gap-2">
              <span className="bg-green-100 text-green-700 w-8 h-8 rounded-full flex items-center justify-center text-sm">1</span>
              貼上文字
            </h2>
            <button 
              onClick={() => setInputText(EXAMPLE_TEXT)}
              className="text-sm text-blue-500 hover:text-blue-700 underline flex items-center gap-1"
            >
              <Copy size={14} /> 帶入範例資料
            </button>
          </div>

          <textarea
            className="w-full h-48 p-4 border border-gray-300 rounded-lg focus:ring-2 focus:ring-green-500 focus:border-transparent transition-all mb-6 font-mono text-sm bg-gray-50"
            placeholder="請在此處貼上從 UberEats 網頁全選複製的文字..."
            value={inputText}
            onChange={(e) => setInputText(e.target.value)}
          ></textarea>

          <div className="flex flex-col md:flex-row justify-between items-center gap-4 bg-gray-50 p-4 rounded-lg">
            <div className="flex items-center gap-3">
              <label className="font-bold text-gray-700 flex items-center gap-2">
                <Calculator size={18} /> 設定折扣 (%):
              </label>
              <input 
                type="number" 
                value={discount} 
                onChange={(e) => setDiscount(e.target.value)}
                className="w-20 p-2 border border-gray-300 rounded text-center font-bold text-lg focus:ring-green-500 focus:border-green-500"
              />
              <span className="text-gray-500 text-sm">(例: 輸入 85 代表 85折)</span>
            </div>

            <button 
              onClick={parseText}
              className="w-full md:w-auto px-8 py-3 bg-green-600 text-white rounded-lg font-bold shadow-md hover:bg-green-700 transform hover:scale-105 transition-all flex items-center justify-center gap-2"
            >
              開始整理 <ChevronDown />
            </button>
          </div>
        </section>

        {calculatedData.length > 0 && (
            <div ref={resultRef} className="space-y-8 animate-fade-in">
                
                {/* Step 2: User Summary with Image Export */}
                <section>
                    <div className="flex flex-col md:flex-row justify-between items-center mb-4 gap-4">
                        <div className="flex items-center gap-4">
                             <h2 className="text-2xl font-bold text-gray-800 flex items-center gap-2">
                                <span className="bg-green-100 text-green-700 w-8 h-8 rounded-full flex items-center justify-center text-sm">2</span>
                                團購人點餐明細
                            </h2>
                            {/* 編輯模式開關按鈕 */}
                            <button 
                                onClick={() => setIsEditMode(!isEditMode)}
                                className={`px-3 py-1.5 rounded-lg border text-sm font-medium flex items-center gap-1 transition-all
                                    ${isEditMode 
                                        ? 'bg-orange-100 text-orange-700 border-orange-200 hover:bg-orange-200' 
                                        : 'bg-white text-gray-500 border-gray-200 hover:bg-gray-50'}`}
                            >
                                {isEditMode ? <><Save size={14} /> 完成修改</> : <><Edit2 size={14} /> 修改數量/單價</>}
                            </button>
                        </div>
                       
                        {/* 匯出圖片按鈕 */}
                        <button 
                            onClick={handleExportImage}
                            disabled={!isHtml2CanvasLoaded}
                            className={`px-4 py-2 bg-blue-100 border border-blue-200 text-blue-800 rounded-lg hover:bg-blue-200 font-medium flex items-center gap-2 transition-colors ${!isHtml2CanvasLoaded ? 'opacity-50 cursor-not-allowed' : ''}`}
                        >
                            {copyFeedback === 'imageExport' ? <Check size={16} /> : <ImageIcon size={16}/>}
                            {isHtml2CanvasLoaded ? '匯出手機版圖片' : '載入中...'}
                        </button>
                    </div>

                    {/* 這個 div 是要被截圖的區域，我們加上白色背景與 padding 確保截圖漂亮 */}
                    <div ref={summaryRef} className="bg-white rounded-xl shadow-lg overflow-hidden border border-gray-200">
                        {/* 截圖時顯示一個標題，讓圖片更完整 */}
                        <div className="bg-green-600 text-white p-4 text-center">
                            <h3 className="text-xl font-bold">UberEats 團購明細</h3>
                            {/* 店家名稱輸入框 */}
                            <input
                                type="text"
                                placeholder="請輸入店家名稱"
                                className="mt-2 bg-green-700 text-white placeholder-green-300 text-center border-none rounded px-2 py-1 outline-none w-2/3 focus:bg-green-800 transition-colors"
                                value={shopName}
                                onChange={(e) => setShopName(e.target.value)}
                            />
                            <p className="text-sm opacity-90 mt-2">折扣: {discount}% | 日期: {new Date().toLocaleDateString()}</p>
                        </div>

                        <div className="overflow-x-auto">
                            <table className="w-full text-left">
                                <thead className="bg-gray-100 text-gray-700 border-b border-gray-200">
                                    <tr>
                                        <th className="px-4 py-2 font-semibold w-1/4">姓名 / 折扣金額</th>
                                        <th className="px-4 py-2 font-semibold">品項內容</th>
                                        <th className="px-4 py-2 font-semibold text-center w-24">份數</th>
                                    </tr>
                                </thead>
                                <tbody className="divide-y divide-gray-100">
                                    {calculatedData.map((person, personIdx) => (
                                    // 雙色視覺設計：奇數行與偶數行背景色不同
                                    <tr key={personIdx} className={`hover:bg-green-50 transition-colors border-b border-gray-100 ${personIdx % 2 === 0 ? 'bg-white' : 'bg-gray-50'}`}>
                                        <td className="px-4 py-3 align-top">
                                            <div className="font-bold text-gray-800 text-lg mb-1">
                                                {person.name || <span className="text-gray-400 italic">(未命名)</span>}
                                            </div>
                                            <div className="text-sm flex items-center gap-2 bg-white inline-block px-2 py-1 rounded border border-gray-200 shadow-sm">
                                                <span className="text-gray-500 line-through">${person.totalOriginal}</span>
                                                <ChevronRight size={14} className="text-gray-400"/>
                                                <span className="text-green-600 font-bold text-base">${person.totalDiscounted}</span>
                                            </div>
                                        </td>
                                        <td className="px-4 py-3 align-top">
                                            {/* 使用 space-y-1 讓行距更緊湊，移除內部分隔線 */}
                                            <div className="space-y-1">
                                                {person.items.map((item, itemIdx) => (
                                                <div key={itemIdx} className="leading-tight py-0.5">
                                                    {/* 將品項名稱、備註、價格、編輯框整合在同一個容器內，利用 inline 排列 */}
                                                    <div className="inline">
                                                        {/* 品項名稱加大：text-lg */}
                                                        <span className="font-bold text-gray-800 text-lg mr-1">
                                                            {item.name} 
                                                        </span>
                                                        
                                                        {/* 備註與品項名稱同行 */}
                                                        {item.options.length > 0 && (
                                                        <span className="text-sm text-gray-500 mr-2">
                                                            ({item.options.join('、')})
                                                        </span>
                                                        )}
                                                    </div>

                                                    {/* 控制項：編輯模式或唯讀模式 */}
                                                    <div className="inline-flex items-center align-baseline">
                                                        {isEditMode ? (
                                                            // 編輯模式：顯示輸入框
                                                            <>
                                                                <div className="inline-flex items-center text-gray-400 text-sm">
                                                                    $
                                                                    <input 
                                                                        type="number"
                                                                        className="w-16 border-b border-gray-300 focus:border-green-500 outline-none bg-transparent text-center text-gray-600"
                                                                        value={item.price}
                                                                        onChange={(e) => handleUpdateItem(person.originalIndex, itemIdx, 'price', e.target.value)}
                                                                        onClick={(e) => e.target.select()}
                                                                    />
                                                                </div>

                                                                <div className="inline-flex items-center ml-1">
                                                                    <span className="text-xs text-gray-500 mr-0.5">x</span>
                                                                    <input 
                                                                        type="number"
                                                                        className="w-10 bg-gray-100 rounded px-1 py-0.5 text-center text-sm font-bold text-gray-700 focus:ring-1 focus:ring-green-500 outline-none"
                                                                        value={item.qty}
                                                                        onChange={(e) => handleUpdateItem(person.originalIndex, itemIdx, 'qty', e.target.value)}
                                                                        onClick={(e) => e.target.select()}
                                                                    />
                                                                </div>
                                                            </>
                                                        ) : (
                                                            // 唯讀模式：顯示純文字
                                                            <>
                                                                <span className="text-xs text-gray-400 font-normal ml-1">${item.price}</span>
                                                                <span className="text-xs bg-gray-200 text-gray-600 px-1.5 py-0.5 rounded-full ml-1">x{item.qty}</span>
                                                            </>
                                                        )}
                                                    </div>
                                                </div>
                                                ))}
                                            </div>
                                        </td>
                                        <td className="px-4 py-3 align-top text-center font-medium">
                                            <span className="inline-block bg-blue-50 text-blue-600 px-3 py-1 rounded-full font-bold">
                                                {person.items.reduce((acc, cur) => acc + cur.qty, 0)}
                                            </span>
                                        </td>
                                    </tr>
                                    ))}
                                </tbody>
                                <tfoot className="bg-gray-50 border-t-2 border-gray-200">
                                    <tr>
                                        <td className="p-4">
                                            <div className="text-xs text-gray-500">總計金額</div>
                                            <div className="flex items-center gap-2 font-bold text-lg">
                                                <span className="text-red-400 line-through text-sm">${calculatedData.reduce((acc, p) => acc + p.totalOriginal, 0).toFixed(0)}</span>
                                                <span className="text-green-600">${calculatedData.reduce((acc, p) => acc + p.totalDiscounted, 0)}</span>
                                            </div>
                                        </td>
                                        <td colSpan="2" className="p-4 text-right align-middle">
                                            <span className="text-gray-600 font-bold mr-2">總份數:</span>
                                            <span className="text-xl font-bold text-blue-600">
                                                {calculatedData.reduce((acc, p) => acc + p.items.reduce((sum, i) => sum + i.qty, 0), 0)}
                                            </span>
                                        </td>
                                    </tr>
                                </tfoot>
                            </table>
                        </div>
                    </div>
                </section>

                {/* Step 3: Item Summary (Grouped by Item, Listed by Variant) */}
                <section>
                    <div className="flex flex-col md:flex-row justify-between items-center mb-4 gap-4">
                        <h2 className="text-2xl font-bold text-gray-800 flex items-center gap-2">
                            <span className="bg-green-100 text-green-700 w-8 h-8 rounded-full flex items-center justify-center text-sm">3</span>
                            餐點統計 (下單用)
                        </h2>
                        <button 
                            onClick={() => copyToClipboard(generateOrderListText(), 'orderList')}
                            className="px-6 py-2 bg-gray-100 border border-gray-300 text-gray-700 rounded-lg hover:bg-gray-200 font-medium shadow-sm flex items-center gap-2"
                        >
                            {copyFeedback === 'orderList' ? <Check size={16} className="text-green-600"/> : <Clipboard size={16}/>}
                            複製訂餐清單
                        </button>
                    </div>

                    <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
                        {aggregatedGroups.map((group, idx) => (
                        <div key={idx} className="bg-white p-5 rounded-xl shadow-md border border-gray-100 hover:shadow-lg transition-shadow">
                            
                            {/* Card Header: Item Name & Total Count */}
                            <div className="flex justify-between items-start mb-3 border-b border-gray-100 pb-2">
                                <div>
                                    <h3 className="text-lg font-bold text-gray-800 leading-tight mb-1">{group.name}</h3>
                                    <p className="text-sm text-gray-500">總金額: ${group.totalPrice.toFixed(2)}</p>
                                </div>
                                <div className="flex items-center justify-center w-10 h-10 bg-blue-50 rounded-full text-blue-600 font-bold text-lg border border-blue-100">
                                    {group.totalCount}
                                </div>
                            </div>
                            
                            {/* Card Body: Variants List */}
                            <div className="space-y-3">
                                {group.variants.map((variant, vIdx) => (
                                    <div key={vIdx} className="bg-gray-50 rounded-lg p-3 text-sm">
                                        <div className="flex justify-between items-start mb-1">
                                            <div className="font-bold text-gray-700">
                                                {variant.options.length > 0 ? variant.options.join('、') : <span className="text-gray-400 font-normal">原味/無備註</span>}
                                            </div>
                                            <span className="bg-white border border-gray-200 px-1.5 rounded text-xs text-gray-600 font-medium">
                                                x{variant.count}
                                            </span>
                                        </div>
                                        
                                        <div className="text-blue-600 font-medium text-xs mt-1">
                                            {variant.buyers.map(b => (
                                                <span key={b.name} className="mr-1">
                                                    {b.name}{b.count > 1 && <span className="text-blue-400">x{b.count}</span>}
                                                    {b !== variant.buyers[variant.buyers.length-1] && '、'}
                                                </span>
                                            ))}
                                        </div>
                                    </div>
                                ))}
                            </div>
                        </div>
                        ))}
                    </div>
                    
                    <div className="mt-8 bg-green-50 p-6 rounded-xl border border-green-100 text-center">
                        <p className="text-gray-600 mb-1">本次團購總數量</p>
                        <p className="text-4xl font-bold text-green-600 mb-4">{aggregatedGroups.reduce((acc, i) => acc + i.totalCount, 0)} <span className="text-lg text-green-500 font-medium">份</span></p>
                        <p className="text-sm text-gray-500">請確認金額與數量無誤後再下單</p>
                    </div>
                </section>
            </div>
        )}

      </main>
    </div>
  );
};

export default App;
