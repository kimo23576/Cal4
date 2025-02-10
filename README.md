<html lang="ar" dir="rtl">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>حاسبة توزيع الأحمال – تحليل تفصيلي لاستقرار المنشآت</title>
  <!-- استدعاء الخطوط وأيقونات Font Awesome -->
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css">
  <!-- خط "Cairo" لتحسين العرض -->
  <link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Cairo:wght@400;600;700&display=swap">
  <!-- مكتبة Chart.js للرسم البياني -->
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <!-- مكتبة AOS لتأثيرات الحركة -->
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/aos@2.3.4/dist/aos.css">
  <!-- مكتبة jsPDF لتصدير النتائج إلى PDF -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
  <!-- مكتبة SheetJS لتصدير النتائج إلى Excel -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
  <style>
    /* إعدادات الخطوط والألوان الأساسية */
    :root {
      --primary-color: #8B4513;
      --secondary-color: #1abc9c;
      --accent-color: #FFD700;
      --text-color: #333;
      --bg-gradient: linear-gradient(45deg, #654321, #8B4513, #A0522D);
    }
    * {
      margin: 0;
      padding: 0;
      box-sizing: border-box;
      scroll-behavior: smooth;
    }
    body {
      font-family: 'Cairo', sans-serif;
      background: var(--bg-gradient);
      background-size: 400% 400%;
      min-height: 100vh;
      position: relative;
      direction: rtl;
      color: var(--text-color);
      transition: background 0.5s, color 0.5s;
    }
    body.light-mode {
      --primary-color: #4a4a4a;
      --secondary-color: #3498db;
      --accent-color: #e67e22;
      --text-color: #333;
      --bg-gradient: linear-gradient(45deg, #f0f0f0, #fff);
      background: var(--bg-gradient);
    }
    @keyframes gradientBG {
      0% { background-position: 0% 50%; }
      50% { background-position: 100% 50%; }
      100% { background-position: 0% 50%; }
    }
    /* شاشة التحميل */
    .loader {
      position: fixed;
      top: 0; left: 0; right: 0; bottom: 0;
      background: var(--bg-gradient);
      display: flex;
      justify-content: center;
      align-items: center;
      z-index: 3000;
    }
    .loader::after {
      content: "";
      width: 50px;
      height: 50px;
      border: 5px solid var(--secondary-color);
      border-top-color: var(--accent-color);
      border-radius: 50%;
      animation: spin 1s linear infinite;
    }
    @keyframes spin { to { transform: rotate(360deg); } }
    
    /* زر تغيير النظام */
    #modeToggle {
      position: fixed;
      top: 20px;
      left: 20px;
      background: var(--secondary-color);
      color: #fff;
      border: none;
      padding: 10px 20px;
      border-radius: 5px;
      z-index: 2100;
      cursor: pointer;
      transition: background 0.3s, transform 0.3s;
    }
    #modeToggle:hover { background: #16a085; transform: scale(1.05); }
    
    /* تنسيق الحاوية الرئيسية */
    .container {
      max-width: 1200px;
      margin: 30px auto;
      padding: 30px;
      background-color: #fff;
      border-radius: 8px;
      box-shadow: 0 0 15px rgba(0,0,0,0.1);
      direction: rtl;
    }
    h1 {
      text-align: center;
      color: var(--primary-color);
      margin-bottom: 20px;
      font-size: 2.5em;
    }
    h2 {
      color: var(--primary-color);
      margin-bottom: 10px;
      border-bottom: 2px solid var(--primary-color);
      padding-bottom: 10px;
      font-size: 1.8em;
    }
    h3 {
      color: var(--secondary-color);
      margin-bottom: 10px;
      font-size: 1.6em;
    }
    label {
      display: block;
      margin-bottom: 8px;
      font-weight: bold;
    }
    input, select, button {
      width: 100%;
      padding: 10px;
      margin-bottom: 15px;
      border: 1px solid #ccc;
      border-radius: 4px;
      font-size: 16px;
    }
    button {
      background-color: var(--primary-color);
      color: #fff;
      cursor: pointer;
      border: none;
      transition: background 0.3s, transform 0.3s;
    }
    button:hover { background-color: var(--accent-color); transform: scale(1.02); }
    .section {
      margin-bottom: 40px;
      padding-bottom: 20px;
      border-bottom: 1px solid #eee;
    }
    .result {
      margin-top: 20px;
      padding: 20px;
      background-color: #ecf0f1;
      border-radius: 4px;
      text-align: center;
      font-size: 1.2em;
    }
    .chart-container { margin-top: 30px; }
    .export-buttons { margin-top: 20px; text-align: center; }
    .export-buttons button { margin: 5px; }
    .reset-button { background-color: #d9534f; margin-bottom: 20px; }
    .reset-button:hover { background-color: #c9302c; }
    
    /* قسم حاسبة توزيع الأحمال */
    .calc-section {
      margin-top: 20px;
      padding: 15px;
      background-color: #f7f7f7;
      border: 1px solid #ddd;
      border-radius: 5px;
    }
    
    /* قسم تحليل السيناريوهات */
    #scenarioResults {
      margin-top: 20px;
      padding: 15px;
      background-color: #f7f7f7;
      border: 1px solid #ddd;
      border-radius: 5px;
    }
    #scenarioComparison {
      margin-top: 20px;
    }
    
    /* الأقسام التعليمية والتوضيحية */
    .info-section {
      margin-top: 40px;
      padding: 20px;
      background-color: #f9f9f9;
      border-radius: 8px;
      border: 1px solid #ddd;
      font-size: 1.1em;
    }
    .info-section h2 { color: var(--primary-color); margin-bottom: 10px; }
    .info-section p { margin-bottom: 10px; color: #555; line-height: 1.6; }
    .info-section ul { list-style-type: disc; margin-right: 20px; color: #555; }
    .info-section ul li { margin-bottom: 5px; }
    
    /* زر العودة للأعلى */
    #backToTop {
      position: fixed;
      bottom: 40px;
      right: 20px;
      background: var(--secondary-color);
      color: #fff;
      border: none;
      padding: 10px;
      border-radius: 50%;
      font-size: 1.5em;
      cursor: pointer;
      display: none;
      z-index: 2000;
      transition: background 0.3s;
    }
    #backToTop:hover { background: #16a085; }
    
    /* تأثير حركي للأزرار العامة */
    .btn-animate {
      transition: transform 0.3s, box-shadow 0.3s;
    }
    .btn-animate:hover {
      transform: scale(1.05);
      box-shadow: 0 5px 15px rgba(0,0,0,0.2);
    }
    
    /* تنسيق جدول السيناريوهات */
    #scenarioComparison table {
      width: 100%;
      border-collapse: collapse;
      margin-top: 15px;
    }
    #scenarioComparison th, #scenarioComparison td {
      border: 1px solid #ccc;
      padding: 10px;
      text-align: center;
    }
    #scenarioComparison th { background-color: var(--primary-color); color: #fff; }
  </style>
</head>
<body>
  <!-- شاشة التحميل -->
  <div id="loader" class="loader"></div>
  
  <div class="container">
    <h1>حاسبة توزيع الأحمال</h1>
    <p style="text-align: center; color: #555;">أداة متقدمة لتحليل توزيع الأحمال على المنشآت لضمان الاستقرار والأمان وفقًا للمعايير الدولية.</p>
    
    <!-- نموذج إدخال البيانات -->
    <div class="section calc-section" id="dataSection" data-aos="fade-up">
      <h2>مدخلات التصميم</h2>
      <p style="color: #555; font-size: 1.1em;">
        أدخل المعطيات التالية لتحليل توزيع الأحمال على عتبة بسيطة (منشأة معروضة على دعامتين). تشمل المدخلات:
        <br>• طول العتبة (L) بالأمتار.
        <br>• الأحمال الأساسية (الحمل الموزع الأساسي) مع إمكانية تطبيق عامل تحميل ديناميكي.
        <br>• إدخال أحمال إضافية (نقطية أو موزعة) عبر الجدول لتفصيل توزيع الأحمال.
      </p>
      
      <label for="beamLength" title="أدخل طول العتبة بالأمتار">طول العتبة (L) بالأمتار:</label>
      <input type="number" id="beamLength" placeholder="مثال: 10" step="0.1">
      
      <label for="uniformLoad" title="أدخل الحمل الموزع الأساسي (kN/m)">الحمل الموزع الأساسي (w) بالكيلو نيوتن/م:</label>
      <input type="number" id="uniformLoad" placeholder="مثال: 5" step="0.1">
      
      <label for="dynamicFactor" title="عامل التحميل الديناميكي لتكبير الأحمال (افتراضي 1)">عامل التحميل الديناميكي:</label>
      <input type="number" id="dynamicFactor" placeholder="مثال: 1" step="0.1" value="1">
      
      <!-- جدول إدخال الأحمال الإضافية -->
      <h3>إدخال الأحمال الإضافية</h3>
      <p style="font-size: 0.9em; color: #555;">يمكنك إدخال حتى 3 أحمال إضافية؛ في حالة الحمل النقطي يتم تجاهل خانة "نهاية الحمل".</p>
      <table style="width:100%; margin-bottom:15px; border:1px solid #ccc; border-collapse: collapse;">
        <thead style="background-color: var(--primary-color); color: #fff;">
          <tr>
            <th style="padding:8px;">#</th>
            <th style="padding:8px;">نوع الحمل</th>
            <th style="padding:8px;">القيمة (kN)</th>
            <th style="padding:8px;">الموقع (م)</th>
            <th style="padding:8px;">نهاية الحمل (م)</th>
          </tr>
        </thead>
        <tbody>
          <tr>
            <td style="padding:8px;">1</td>
            <td style="padding:8px;">
              <select id="loadType1">
                <option value="point" selected>نقطة</option>
                <option value="distributed">موزعة</option>
              </select>
            </td>
            <td style="padding:8px;"><input type="number" id="loadValue1" placeholder="مثال: 20" step="0.1"></td>
            <td style="padding:8px;"><input type="number" id="loadPosition1" placeholder="مثال: 4" step="0.1"></td>
            <td style="padding:8px;"><input type="number" id="loadEnd1" placeholder="---" step="0.1" disabled></td>
          </tr>
          <tr>
            <td style="padding:8px;">2</td>
            <td style="padding:8px;">
              <select id="loadType2">
                <option value="point">نقطة</option>
                <option value="distributed" selected>موزعة</option>
              </select>
            </td>
            <td style="padding:8px;"><input type="number" id="loadValue2" placeholder="مثال: 5" step="0.1"></td>
            <td style="padding:8px;"><input type="number" id="loadPosition2" placeholder="بداية" step="0.1"></td>
            <td style="padding:8px;"><input type="number" id="loadEnd2" placeholder="نهاية" step="0.1"></td>
          </tr>
          <tr>
            <td style="padding:8px;">3</td>
            <td style="padding:8px;">
              <select id="loadType3">
                <option value="point" selected>نقطة</option>
                <option value="distributed">موزعة</option>
              </select>
            </td>
            <td style="padding:8px;"><input type="number" id="loadValue3" placeholder="مثال: 15" step="0.1"></td>
            <td style="padding:8px;"><input type="number" id="loadPosition3" placeholder="مثال: 7" step="0.1"></td>
            <td style="padding:8px;"><input type="number" id="loadEnd3" placeholder="---" step="0.1" disabled></td>
          </tr>
        </tbody>
      </table>
      
      <button onclick="calculateLoadDistribution()" class="btn-animate"><i class="fas fa-calculator"></i> احسب توزيع الأحمال</button>
      <button class="reset-button btn-animate" onclick="resetLoadCalculator()"><i class="fas fa-undo"></i> إعادة تعيين</button>
    </div>
    
    <!-- عرض النتائج -->
    <div class="result" id="loadResultSection" data-aos="fade-up">
      <h2>نتائج توزيع الأحمال</h2>
      <p id="reactionLeft"></p>
      <p id="reactionRight"></p>
      <p id="maxShear"></p>
      <p id="maxMoment"></p>
      <div class="export-buttons">
        <button onclick="printResult()" class="btn-animate"><i class="fas fa-print"></i> طباعة</button>
        <button onclick="exportPDF()" class="btn-animate"><i class="fas fa-file-pdf"></i> مشاركة النتيجة</button>
        <button onclick="exportExcel()" class="btn-animate"><i class="fas fa-file-excel"></i> تصدير إلى Excel</button>
      </div>
    </div>
    
    <!-- رسم بياني لتوزيع الجرف والعزم -->
    <div class="chart-container" data-aos="fade-up">
      <canvas id="loadChart"></canvas>
    </div>
    
    <!-- قسم تحليل السيناريوهات -->
    <div class="section" data-aos="fade-up">
      <h2>تحليل السيناريوهات</h2>
      <p style="color: #555;">يمكنك تجربة نسب تحميل مختلفة لتعديل قيمة الأحمال مؤقتاً (مثلاً 80%، 100%، 120%)، ثم حفظ السيناريو الحالي وعرض جدول مقارنة مفصل لجميع السيناريوهات لتقييم تأثير التغييرات على ردود الفعل والمخططات.</p>
      <div style="display:flex; gap:10px; margin-bottom:20px;">
        <button onclick="calculateScenario(0.8)" class="btn-animate"><i class="fas fa-percentage"></i> 80%</button>
        <button onclick="calculateScenario(1)" class="btn-animate"><i class="fas fa-percentage"></i> 100%</button>
        <button onclick="calculateScenario(1.2)" class="btn-animate"><i class="fas fa-percentage"></i> 120%</button>
      </div>
      <div id="scenarioResults"></div>
      <button onclick="saveScenario()" class="btn-animate" style="margin-top:10px;"><i class="fas fa-save"></i> حفظ السيناريو الحالي</button>
      <div id="scenarioComparison"></div>
    </div>
    
    <!-- قسم دليل المستخدم التفصيلي -->
    <div class="info-section" data-aos="fade-up">
      <h2>دليل المستخدم - شرح تفصيلي لتوزيع الأحمال</h2>
      <p>تستخدم هذه الحاسبة لتحليل توزيع الأحمال على عتبة بسيطة لضمان استقرار المنشأة وسلامتها. فيما يلي شرح مفصل للخطوات والمعادلات المستخدمة:</p>
      <ul>
        <li><strong>إدخال المعطيات:</strong> تُدخل معطيات العتبة الأساسية مثل طول العتبة (L) والحمل الموزع الأساسي (w). كما يمكنك إدخال عامل تحميل ديناميكي لتعديل الأحمال (مثلاً 80%، 100%، 120%).</li>
        <li><strong>إدخال الأحمال الإضافية:</strong> يُمكن إدخال حتى 3 أحمال إضافية، حيث يتم تحديد نوع كل حمل (نقطة أو موزعة)، وقيمته وموقعه. في حالة الحمل النقطي يتم تجاهل خانة "نهاية الحمل".</li>
        <li><strong>حساب ردود الفعل:</strong> باستخدام معادلات التوازن:
          <ul>
            <li>∑F = 0: مجموع الأحمال = Rₗ + Rᵣ.</li>
            <li>∑M حول الطرف الأيسر = 0: Rᵣ = (مجموع (P × المسافة من الطرف)) ÷ L، ثم Rₗ = إجمالي الحمل - Rᵣ.</li>
          </ul>
        </li>
        <li><strong>حساب مخططات الجرف والعزم:</strong> تُحسب الجرف عند كل نقطة على العتبة باستخدام: V(x) = Rₗ – (مجموع الأحمال حتى x)، ويتم حساب العزم باستخدام: M(x) = Rₗ × x – (مجموع (P × (x – موقع الحمل))).</li>
        <li><strong>رسم المخططات:</strong> تُعرض مخططات الجرف والعزم باستخدام Chart.js لتوضيح توزيع القوى وتأثيرها على استقرار المنشأة.</li>
        <li><strong>تحليل السيناريوهات:</strong> يمكنك تعديل الأحمال بنسبة معينة (عن طريق عامل التحميل الديناميكي) ثم حفظ السيناريو الحالي وعرض جدول مقارنة مفصل لجميع السيناريوهات المحفوظة، مما يساعد على تقييم تأثير التغييرات.</li>
        <li><strong>التصدير والطباعة:</strong> تتوفر خيارات لتصدير النتائج إلى PDF أو Excel أو طباعتها مباشرةً.</li>
      </ul>
      <p>تُعد هذه الحاسبة أداة تقديرية تُساعد مهندسي التصميم في الحصول على فكرة أولية عن توزيع الأحمال وردود الفعل، ويُنصح باستخدامها مع برامج التصميم المتقدمة لإجراء التحليلات التفصيلية.</p>
    </div>
    
    <div class="info-section" data-aos="fade-up">
      <h2>المعايير الدولية وأهمية توزيع الأحمال</h2>
      <p>يُعتمد في تصميم المنشآت على معادلات توزيع الأحمال وفقًا للمعايير الدولية لضمان استقرار المنشأة وسلامتها. يساهم التحليل التفصيلي لتوزيع الأحمال في تقليل المخاطر وتحسين الأداء الهيكلي للمنشآت.</p>
    </div>
    
  </div>
  
  <!-- زر العودة للأعلى -->
  <button id="backToTop" title="العودة للأعلى" aria-label="العودة للأعلى">↑</button>
  
  <!-- الفوتر (يظهر مرة واحدة فقط) -->
  <footer style="background: #333; color: #ccc; padding: 20px; text-align: center; direction: rtl; margin-top: 40px;">
    <p style="margin-bottom: 10px;">© 2024 منصة الاتحاد العام للمقاولين اليمنيين - جميع الحقوق محفوظة</p>
    <p>
      <a href="https://guyc-yemen.com/" style="color: var(--accent-color); text-decoration: none;">الموقع الرسمي</a> |
      <a href="mailto:info@guyc-ye.com" style="color: var(--accent-color); text-decoration: none;">البريد الإلكتروني</a> |
      <a href="tel:01450553" style="color: var(--accent-color); text-decoration: none;">01-450553</a>
    </p>
    <p style="margin-top: 10px;">
      <a href="#" style="color: var(--accent-color); margin: 0 5px; text-decoration: none;"><i class="fab fa-facebook"></i></a>
      <a href="#" style="color: var(--accent-color); margin: 0 5px; text-decoration: none;"><i class="fab fa-twitter"></i></a>
      <a href="#" style="color: var(--accent-color); margin: 0 5px; text-decoration: none;"><i class="fab fa-youtube"></i></a>
      <a href="#" style="color: var(--accent-color); margin: 0 5px; text-decoration: none;"><i class="fab fa-whatsapp"></i></a>
    </p>
  </footer>
  
  <!-- استدعاء مكتبة AOS -->
  <script src="https://cdn.jsdelivr.net/npm/aos@2.3.4/dist/aos.js"></script>
  <script>
    window.addEventListener('load', () => {
      AOS.init();
      document.getElementById('loader').style.display = 'none';
      loadInputs();
    });
  </script>
  
  <!-- دوال حاسبة توزيع الأحمال -->
  <script>
    // حفظ البيانات في localStorage
    function saveInputs() {
      const inputs = {
        projectName: document.getElementById('projectName').value,
        totalCost: document.getElementById('totalCost').value,
        exchangeRate: document.getElementById('exchangeRate').value,
        buildingType: document.getElementById('buildingType').value,
        buildingUsage: document.getElementById('buildingUsage').value,
        financingType: document.getElementById('financingType').value,
        financingPercent: document.getElementById('financingPercent').value,
        annualRevenue: document.getElementById('annualRevenue').value,
        annualExpenses: document.getElementById('annualExpenses').value,
        interestRate: document.getElementById('interestRate').value,
        projectDuration: document.getElementById('projectDuration').value
      };
      localStorage.setItem('engCalcInputs', JSON.stringify(inputs));
    }
    
    // استعادة البيانات من localStorage
    function loadInputs() {
      const saved = localStorage.getItem('engCalcInputs');
      if (saved) {
        const inputs = JSON.parse(saved);
        document.getElementById('projectName').value = inputs.projectName || "";
        document.getElementById('totalCost').value = inputs.totalCost || "";
        document.getElementById('exchangeRate').value = inputs.exchangeRate || "250";
        document.getElementById('buildingType').value = inputs.buildingType || "residential";
        document.getElementById('buildingUsage').value = inputs.buildingUsage || "residential";
        document.getElementById('financingType').value = inputs.financingType || "bankLoan";
        document.getElementById('financingPercent').value = inputs.financingPercent || "30";
        document.getElementById('annualRevenue').value = inputs.annualRevenue || "";
        document.getElementById('annualExpenses').value = inputs.annualExpenses || "";
        document.getElementById('interestRate').value = inputs.interestRate || "8";
        document.getElementById('projectDuration').value = inputs.projectDuration || "10";
      }
    }
    
    // زر تحديث سعر الصرف من API (إن وجد)
    function updateExchangeRate() {
      fetch("https://api.exchangerate-api.com/v4/latest/USD")
        .then(response => response.json())
        .then(data => {
          if(data && data.rates && data.rates["YER"]) {
            const rate = data.rates["YER"];
            document.getElementById("exchangeRate").value = rate;
            alert("تم تحديث سعر الصرف: " + rate);
            saveInputs();
          } else {
            alert("تعذر الحصول على سعر الصرف.");
          }
        })
        .catch(err => alert("فشل تحديث سعر الصرف: " + err));
    }
    
    // استخدام debounce لتحديث النتائج تلقائيًا
    const inputFields = document.querySelectorAll('#dataSection input, #dataSection select');
    let debounceTimer;
    inputFields.forEach(field => {
      field.addEventListener('input', () => {
        clearTimeout(debounceTimer);
        debounceTimer = setTimeout(() => {
          calculateLoadDistribution();
          saveInputs();
        }, 500);
      });
    });
    
    // دالة حساب توزيع الأحمال
    function calculateLoadDistribution() {
      // قراءة معطيات العتبة
      const L = parseFloat(document.getElementById('beamLength').value); // طول العتبة (م)
      let w = parseFloat(document.getElementById('uniformLoad').value);   // الحمل الموزع الأساسي (kN/m)
      const dynamicFactor = parseFloat(document.getElementById('dynamicFactor').value) || 1;
      // تطبيق عامل التحميل الديناميكي
      w = w * dynamicFactor;
      
      if (!L || L <= 0) return;
      
      // تعريف مصفوفة لتخزين الأحمال الإضافية من الجدول
      let loads = [];
      
      // دالة مساعدة لحساب الحمل المكافئ (نقطة أو موزعة)
      function getLoad(loadType, value, pos, end) {
        if (loadType === "point") {
          return { P: parseFloat(value) || 0, a: parseFloat(pos) || 0 };
        } else if (loadType === "distributed") {
          let start = parseFloat(pos) || 0;
          let finish = parseFloat(end) || start;
          let total = (parseFloat(value) || 0) * (finish - start);
          let mid = (start + finish) / 2;
          return { P: total, a: mid };
        }
        return { P: 0, a: 0 };
      }
      
      // قراءة الأحمال الإضافية من الجدول (3 صفوف)
      let load1 = getLoad(
        document.getElementById('loadType1').value,
        document.getElementById('loadValue1').value,
        document.getElementById('loadPosition1').value,
        document.getElementById('loadEnd1').value
      );
      let load2 = getLoad(
        document.getElementById('loadType2').value,
        document.getElementById('loadValue2').value,
        document.getElementById('loadPosition2').value,
        document.getElementById('loadEnd2').value
      );
      let load3 = getLoad(
        document.getElementById('loadType3').value,
        document.getElementById('loadValue3').value,
        document.getElementById('loadPosition3').value,
        document.getElementById('loadEnd3').value
      );
      loads.push(load1, load2, load3);
      
      // حساب إجمالي الحمل ومجموع العزوم حول الطرف الأيسر
      let totalLoad = 0, sumMoment = 0;
      loads.forEach(load => {
        totalLoad += load.P;
        sumMoment += load.P * load.a;
      });
      
      // باستخدام معادلات التوازن:
      // رد فعل اليمين (Rᵣ) = (مجموع (P × a)) ÷ L
      // رد فعل اليسار (Rₗ) = إجمالي الحمل - Rᵣ
      let Rr = sumMoment / L;
      let Rl = totalLoad - Rr;
      
      // حساب مخططات الجرف والعزم عند نقاط متفرقة على العتبة
      let numPoints = 100;
      let shearValues = [];
      let momentValues = [];
      for (let i = 0; i <= numPoints; i++) {
        let x = (L / numPoints) * i;
        let shear = Rl;
        loads.forEach(load => {
          if (load.a <= x) {
            shear -= load.P;
          }
        });
        shearValues.push(shear);
        
        let moment = Rl * x;
        loads.forEach(load => {
          if (load.a <= x) {
            moment -= load.P * (x - load.a);
          }
        });
        momentValues.push(moment);
      }
      
      // تحديث عرض النتائج
      document.getElementById('reactionLeft').innerText = "رد فعل اليسار (Rₗ): " + Rl.toFixed(2) + " kN";
      document.getElementById('reactionRight').innerText = "رد فعل اليمين (Rᵣ): " + Rr.toFixed(2) + " kN";
      let maxShear = Math.max(...shearValues.map(Math.abs));
      let maxMoment = Math.max(...momentValues.map(Math.abs));
      document.getElementById('maxShear').innerText = "أقصى جرف: " + maxShear.toFixed(2) + " kN";
      document.getElementById('maxMoment').innerText = "أقصى عزم: " + maxMoment.toFixed(2) + " kN·m";
      
      // تحديث الرسم البياني باستخدام Chart.js
      updateLoadChart(L, shearValues, momentValues);
      
      // تحديث قسم السيناريو الأساسي (عرض النتائج الأساسية)
      updateScenarioResult(
        "Rₗ: " + Rl.toFixed(2) + " kN",
        "Rᵣ: " + Rr.toFixed(2) + " kN",
        "أقصى جرف: " + maxShear.toFixed(2) + " kN",
        "أقصى عزم: " + maxMoment.toFixed(2) + " kN·m",
        "السيناريو الأساسي"
      );
    }
    
    // تحديث الرسم البياني لتوزيع الجرف والعزم باستخدام Chart.js
    let loadChartInstance;
    function updateLoadChart(L, shearValues, momentValues) {
      const ctx = document.getElementById('loadChart').getContext('2d');
      let labels = [];
      let numPoints = shearValues.length;
      for (let i = 0; i < numPoints; i++) {
        let x = (L / (numPoints - 1)) * i;
        labels.push(x.toFixed(2) + " م");
      }
      if (loadChartInstance) {
        loadChartInstance.data.labels = labels;
        loadChartInstance.data.datasets[0].data = shearValues;
        loadChartInstance.data.datasets[1].data = momentValues;
        loadChartInstance.update();
      } else {
        loadChartInstance = new Chart(ctx, {
          type: 'line',
          data: {
            labels: labels,
            datasets: [
              {
                label: "مخطط الجرف (kN)",
                data: shearValues,
                backgroundColor: 'rgba(231,76,60,0.3)',
                borderColor: 'rgba(231,76,60,1)',
                borderWidth: 2,
                fill: false,
                tension: 0.3
              },
              {
                label: "مخطط العزم (kN·m)",
                data: momentValues,
                backgroundColor: 'rgba(26,188,156,0.3)',
                borderColor: 'rgba(26,188,156,1)',
                borderWidth: 2,
                fill: false,
                tension: 0.3
              }
            ]
          },
          options: {
            responsive: true,
            animation: { duration: 1500, easing: 'easeOutQuart' },
            scales: {
              y: { beginAtZero: true },
              x: { title: { display: true, text: "المسافة على العتبة (م)" } }
            }
          }
        });
      }
    }
    
    // متغير لتخزين معامل الحمل الحالي في السيناريو (افتراضي 1)
    var currentLoadFactor = 1;
    
    // دالة تحليل السيناريوهات: تعديل الحمل الموزع مؤقتاً وإعادة الحساب
    function calculateScenario(loadFactor) {
      currentLoadFactor = loadFactor;
      let baseUniformLoad = parseFloat(document.getElementById('uniformLoad').value);
      let modifiedLoad = baseUniformLoad * loadFactor;
      // حفظ القيمة الأصلية ثم تعديلها مؤقتاً
      document.getElementById('uniformLoad').value = modifiedLoad;
      calculateLoadDistribution();
      let scenarioText = `<h3>سيناريو تحميل بنسبة ${loadFactor * 100}%</h3>
        <p><strong>${document.getElementById('reactionLeft').innerText}</strong></p>
        <p><strong>${document.getElementById('reactionRight').innerText}</strong></p>
        <p><strong>${document.getElementById('maxShear').innerText}</strong></p>
        <p><strong>${document.getElementById('maxMoment').innerText}</strong></p>`;
      document.getElementById('scenarioResults').innerHTML = scenarioText;
      // استعادة القيمة الأصلية
      document.getElementById('uniformLoad').value = baseUniformLoad;
    }
    
    // تحديث عرض نتائج السيناريو الأساسي
    function updateScenarioResult(RlText, RrText, maxShearText, maxMomentText, scenarioName) {
      const scenarioDiv = document.getElementById('scenarioResults');
      scenarioDiv.innerHTML = `
        <h3>${scenarioName}</h3>
        <p><strong>${RlText}</strong></p>
        <p><strong>${RrText}</strong></p>
        <p><strong>${maxShearText}</strong></p>
        <p><strong>${maxMomentText}</strong></p>
      `;
    }
    
    // حفظ السيناريو الحالي للمقارنة
    let savedScenarios = [];
    function saveScenario() {
      const scenario = {
        projectName: document.getElementById('projectName').value || "المشروع غير محدد",
        loadFactor: currentLoadFactor,
        reactionLeft: document.getElementById('reactionLeft').innerText,
        reactionRight: document.getElementById('reactionRight').innerText,
        maxShear: document.getElementById('maxShear').innerText,
        maxMoment: document.getElementById('maxMoment').innerText,
        timestamp: new Date().toLocaleString()
      };
      savedScenarios.push(scenario);
      updateScenarioComparison();
    }
    
    // عرض مقارنة السيناريوهات المحفوظة
    function updateScenarioComparison() {
      const compDiv = document.getElementById('scenarioComparison');
      if (savedScenarios.length === 0) {
        compDiv.innerHTML = "<p>لا توجد سيناريوهات محفوظة.</p>";
        return;
      }
      let tableHTML = `<table>
                          <thead>
                            <tr style="background-color: var(--primary-color); color: #fff;">
                              <th style="padding: 10px;">التاريخ</th>
                              <th style="padding: 10px;">نسبة التحميل</th>
                              <th style="padding: 10px;">Rₗ</th>
                              <th style="padding: 10px;">Rᵣ</th>
                              <th style="padding: 10px;">أقصى جرف</th>
                              <th style="padding: 10px;">أقصى عزم</th>
                            </tr>
                          </thead>
                          <tbody>`;
      savedScenarios.forEach(scn => {
        tableHTML += `<tr>
                        <td style="padding: 10px;">${scn.timestamp}</td>
                        <td style="padding: 10px;">${scn.loadFactor * 100}%</td>
                        <td style="padding: 10px;">${scn.reactionLeft}</td>
                        <td style="padding: 10px;">${scn.reactionRight}</td>
                        <td style="padding: 10px;">${scn.maxShear}</td>
                        <td style="padding: 10px;">${scn.maxMoment}</td>
                      </tr>`;
      });
      tableHTML += `</tbody></table>`;
      compDiv.innerHTML = `<h3>مقارنة السيناريوهات المحفوظة</h3>` + tableHTML;
    }
    
    // تصدير النتائج إلى PDF باستخدام jsPDF
    function exportPDF() {
      const { jsPDF } = window.jspdf;
      const doc = new jsPDF("p", "mm", "a4");
      doc.setFont("Arial", "normal");
      doc.text("تقرير حاسبة توزيع الأحمال", 20, 20);
      const results = document.getElementById("loadResultSection").innerText;
      doc.text(results, 20, 40);
      doc.text("يرجى مراجعة باقي الأقسام للحصول على التفاصيل الكاملة.", 20, 60);
      doc.save("تقرير_توزيع_الأحمال.pdf");
    }
    
    // تصدير النتائج إلى Excel باستخدام SheetJS
    function exportExcel() {
      let data = [
        ["Rₗ", document.getElementById('reactionLeft').innerText],
        ["Rᵣ", document.getElementById('reactionRight').innerText],
        ["أقصى جرف", document.getElementById('maxShear').innerText],
        ["أقصى عزم", document.getElementById('maxMoment').innerText]
      ];
      let ws = XLSX.utils.aoa_to_sheet(data);
      let wb = XLSX.utils.book_new();
      XLSX.utils.book_append_sheet(wb, ws, "النتائج");
      XLSX.writeFile(wb, "تقرير_توزيع_الأحمال.xlsx");
    }
    
    // دالة الطباعة
    function printResult() {
      window.print();
    }
    
    // دالة إعادة تعيين الحقول لحاسبة توزيع الأحمال
    function resetLoadCalculator() {
      document.getElementById('projectName').value = "";
      document.getElementById('totalCost').value = "";
      document.getElementById('exchangeRate').value = "250";
      document.getElementById('buildingType').selectedIndex = 0;
      document.getElementById('buildingUsage').selectedIndex = 0;
      document.getElementById('financingType').selectedIndex = 0;
      document.getElementById('financingPercent').value = "30";
      document.getElementById('annualRevenue').value = "";
      document.getElementById('annualExpenses').value = "";
      document.getElementById('interestRate').value = "8";
      document.getElementById('projectDuration').value = "10";
      // مدخلات توزيع الأحمال
      document.getElementById('beamLength').value = "";
      document.getElementById('uniformLoad').value = "";
      document.getElementById('dynamicFactor').value = "1";
      if(document.getElementById('beamType')) {
        document.getElementById('beamType').selectedIndex = 0;
      }
      // إعادة تعيين بيانات الجدول (الأحمال)
      document.getElementById('loadValue1').value = "";
      document.getElementById('loadPosition1').value = "";
      document.getElementById('loadEnd1').value = "";
      document.getElementById('loadValue2').value = "";
      document.getElementById('loadPosition2').value = "";
      document.getElementById('loadEnd2').value = "";
      document.getElementById('loadValue3').value = "";
      document.getElementById('loadPosition3').value = "";
      document.getElementById('loadEnd3').value = "";
      document.getElementById('loadResultSection').innerHTML = "<h2>نتائج توزيع الأحمال</h2>";
      if(loadChartInstance) {
        loadChartInstance.destroy();
      }
      document.getElementById('scenarioResults').innerHTML = "";
      document.getElementById('scenarioComparison').innerHTML = "";
      savedScenarios = [];
      saveInputs();
    }
  </script>
