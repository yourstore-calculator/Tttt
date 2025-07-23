# doors and window
<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>حاسبة الأسعار</title>
  <style>
    body {
      font-family: 'Arial', sans-serif;
      background-color: #f7f7f7;
      color: #333;
      padding: 20px;
    }
    h1 {
      text-align: center;
      color: #444;
    }
    .container {
      max-width: 900px;
      margin: auto;
      background: white;
      padding: 20px;
      border-radius: 8px;
      box-shadow: 0 0 15px rgba(0,0,0,0.1);
    }
    label {
      font-weight: bold;
    }
    select, input {
      width: 100%;
      padding: 8px;
      margin: 5px 0 15px;
    }
    button {
      padding: 10px 20px;
      background-color: #222;
      color: white;
      border: none;
      cursor: pointer;
      margin-top: 10px;
    }
    #results {
      margin-top: 30px;
      background: #eee;
      padding: 15px;
      border-radius: 6px;
    }
    .hidden {
      display: none;
    }
    .result-item {
      background: white;
      border: 1px solid #ddd;
      padding: 10px;
      margin-bottom: 10px;
    }
  </style>
</head>
<body>
  <div class="container">
    <h1>حاسبة الأسعار</h1>

    <label>نوع المنتج</label>
    <select id="category" onchange="updateTypes()">
      <option value="">-- اختر القسم --</option>
      <option value="نوافذ">نوافذ</option>
      <option value="أبواب">أبواب</option>
      <option value="أبواب السحب">أبواب السحب</option>
    </select>

    <label>النوع</label>
    <select id="type" onchange="toggleAddons()"></select>

    <label>الطول (متر)</label>
    <input type="number" id="length" value="2.2" step="0.01" />

    <label>العرض (متر)</label>
    <input type="number" id="width" value="1" step="0.01" />

    <label>الكمية</label>
    <input type="number" id="quantity" value="1" min="1"/>

    <div id="addons" class="hidden">
      <label><input type="checkbox" id="addCurtain" onchange="toggleCurtainSize()" /> إضافة ستارة داخلية</label><br>
      <label><input type="checkbox" id="addMesh" /> إضافة شبك</label>
    </div>

    <div id="curtainSize" class="hidden">
      <label>مقاس الستارة (الطول × العرض):</label>
      <input type="number" id="curtainLength" value="2.2" step="0.01" />
      <input type="number" id="curtainWidth" value="1" step="0.01" />
    </div>

    <button onclick="calculate()">احسب</button>
    <button onclick="clearResults()">🗑️ مسح النتائج</button>

    <div id="results"></div>
  </div>

  <script>
    const typesData = {
      "نوافذ": {
        "دبل جلاس دبل فريم - ثابتة": { price: 34, factor: 0.13 },
        "دبل جلاس دبل فريم - حركة": { price: 73, factor: 0.13 },
        "دبل جلاس دبل فريم - حركتين": { price: 92, factor: 0.13 },

        "دبل جلاس سنجل فريم - ثابتة": { price: 26, factor: 0.13 },
        "دبل جلاس سنجل فريم - حركة": { price: 46, factor: 0.13 },
        "دبل جلاس سنجل فريم - حركتين": { price: 58, factor: 0.13 },

        "سنجل جلاس سنجل فريم - ثابتة": { price: 20, factor: 0.13 },
        "سنجل جلاس سنجل فريم - حركة": { price: 43, factor: 0.13 },
        "سنجل جلاس سنجل فريم - حركتين": { price: 47, factor: 0.13 },

        "نوافذ السلايدنج": { price: 0, factor: 0.13 },
        "النوافذ الكهربائية": { price: 102, factor: 0.13 },
        "سكاي لايت - بدون مكينة": { price: 56, factor: 0.13 },
        "سكاي لايت - مع مكينة": { price: 145, factor: 0.13 },
        "كارتن وول - ثقيل": { price: 56, factor: 0.13 },
        "كارتن وول - خفيف": { price: 45, factor: 0.13 }
      },
      "أبواب": {
        "باب مدخل - زينك": { price: 66 + 10, factor: 0.2 },
        "باب مدخل - ستينلس ستيل": { price: 120 + 10, factor: 0.2 },
        "باب مدخل - كاست المنيوم": { price: 168 + 10, factor: 0.2 },

        "باب WPC - فارغ": { price: 45, factor: 0.11 },
        "باب WPC - مع خشب": { price: 50, factor: 0.11 },
        "باب WPC - ضد الصوت": { price: 60, factor: 0.11 },
        "باب WPC - مع فريم ألمنيوم": { price: 67, factor: 0.11 },

        "باب ألمنيوم - فارغ": { price: 65, factor: 0.11 },
        "باب ألمنيوم - مع خشب": { price: 75, factor: 0.11 },
        "باب ألمنيوم - فل ألمنيوم": { price: 85, factor: 0.11 },
        "باب ألمنيوم - مخفي": { price: 110, factor: 0.11 },
        "باب ألمنيوم - خارجي": { price: 61, factor: 0.11 },

        "باب دورات مياه - جديد": { price: 55, factor: 0.11 },
        "باب دورات مياه - قديم": { price: 45, factor: 0.11 },
        "باب دورات مياه - مخفي زجاجي": { price: 65, factor: 0.11 },

        "باب حديقة": { price: 91, factor: 0.2 }
      },
      "أبواب السحب": {
        "داخلي زجاج": { price: 38, factor: 0.13 },
        "داخلي متين": { price: 41, factor: 0.13 },
        "خارجي - جزء مفتوح": { price: 55, factor: 0.13 },
        "خارجي - جزئين": { price: 58, factor: 0.13 },
        "WPC سلايد": { price: 61, factor: 0.11 },
        "فولدنج داخلي": { price: 39, factor: 0.13 },
        "فولدنج خارجي": { price: 56, factor: 0.13 },
        "شتر دور خارجي": { price: 28, factor: 0.13 }
      }
    };

    const addonsEligibleTypes = [
      "دبل جلاس", "نوافذ السلايدنج", "باب", "سحب", "كارتن", "سكاي"
    ];

    function updateTypes() {
      const category = document.getElementById("category").value;
      const typeSelect = document.getElementById("type");
      typeSelect.innerHTML = "";

      if (typesData[category]) {
        Object.keys(typesData[category]).forEach(type => {
          const option = document.createElement("option");
          option.value = type;
          option.textContent = type;
          typeSelect.appendChild(option);
        });
      }
    }

    function toggleAddons() {
      const type = document.getElementById("type").value;
      const addons = document.getElementById("addons");

      const showAddons = addonsEligibleTypes.some(keyword => type.includes(keyword));
      addons.classList.toggle("hidden", !showAddons);
    }

    function toggleCurtainSize() {
      document.getElementById("curtainSize").classList.toggle(
        "hidden",
        !document.getElementById("addCurtain").checked
      );
    }

    function calculate() {
      const category = document.getElementById("category").value;
      const type = document.getElementById("type").value;
      const length = parseFloat(document.getElementById("length").value);
      const width = parseFloat(document.getElementById("width").value);
      const quantity = parseInt(document.getElementById("quantity").value);
      const curtain = document.getElementById("addCurtain").checked;
      const mesh = document.getElementById("addMesh").checked;

      const area = length * width;
      const data = typesData[category][type];
      let unitPrice;

      if (type.includes("نوافذ السلايدنج")) {
        unitPrice = (area * 80) + 10; // مثال: سعر متغير
      } else {
        unitPrice = area * data.price;
      }

      const shipping = area * data.factor * 48;
      let total = (unitPrice + shipping) * quantity;

      let details = `
        <div class="result-item">
          <strong>النوع:</strong> ${type}<br>
          <strong>المقاس:</strong> ${length} × ${width} م<br>
          <strong>الكمية:</strong> ${quantity}<br>
          <strong>سعر الشحن:</strong> ${shipping.toFixed(2)} ريال<br>
          <strong>السعر الكلي:</strong> ${total.toFixed(2)} ريال
      `;

      if (curtain) {
        const cLength = parseFloat(document.getElementById("curtainLength").value);
        const cWidth = parseFloat(document.getElementById("curtainWidth").value);
        const curtainPrice = cLength * cWidth * 26;
        details += `<br><strong>ستارة:</strong> ${curtainPrice.toFixed(2)} ريال`;
        total += curtainPrice;
      }

      if (mesh) {
        const meshPrice = area * 14; // مثال لشبك السلايد
        details += `<br><strong>شبك:</strong> ${meshPrice.toFixed(2)} ريال`;
        total += meshPrice;
      }

      details += `<br><strong>الإجمالي النهائي:</strong> ${total.toFixed(2)} ريال</div>`;

      document.getElementById("results").innerHTML += details;
    }

    function clearResults() {
      document.getElementById("results").innerHTML = "";
    }
  </script>
</body>
</html>
