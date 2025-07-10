<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
  <meta charset="UTF-8">
  <title>معادلة Albasatneh لاستخراج جذور دالة زيتا</title>
  <script src="https://polyfill.io/v3/polyfill.min.js?features=es6"></script>
  <script id="MathJax-script" async
    src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js"></script>
  <style>
    body {
      background-color: #000;
      color: #fff;
      font-family: 'Arial', sans-serif;
      padding: 30px;
      line-height: 1.8;
    }
    h1, h2 {
      color: #00ffff;
    }
    .equation {
      color: #ffcc00;
      font-size: 20px;
      margin: 30px 0;
    }
    .section {
      margin-top: 40px;
    }
  </style>
</head>
<body>

  <h1>معادلة Albasatneh لاستخراج الجذور غير البديهية لدالة زيتا</h1>

  <div class="section">
    <p><strong>الاسم:</strong> Alaa Sheikh Albasatneh</p>
    <p><strong>الجنسية:</strong> سوري</p>
    <p><strong>التاريخ:</strong> 10 يوليو 2025</p>
  </div>

  <div class="section">
    <h2>المعادلة الرسمية:</h2>
    <div class="equation">
      \[
      t_k = \frac{2\theta k + C_0 + \sum_{n=0}^{8} \beta_n \left( \frac{\ln k}{\ln \ln k} \right)^n}{f}
      \]
    </div>
  </div>

  <div class="section">
    <h2>الثوابت المعتمدة:</h2>
    <ul>
      <li>\( f = 1 \)</li>
      <li>\( C_0 = -6.180555 \)</li>
      <li>\( c = 0.844327 \) (سعة الجيبية)</li>
      <li>كثيرة الحدود التصحيحية \( \beta_n \) حتى الدرجة 8:</li>
      <ul>
        <li>\( \beta_0 = 0.774963 \)</li>
        <li>\( \beta_1 = -0.225223 \)</li>
        <li>\( \beta_2 = 0.053304 \)</li>
        <li>\( \beta_3 = -0.010113 \)</li>
        <li>\( \beta_4 = 0.001562 \)</li>
        <li>\( \beta_5 = -0.000200 \)</li>
        <li>\( \beta_6 = 0.000020 \)</li>
        <li>\( \beta_7 = -0.000002 \)</li>
        <li>\( \beta_8 = 0.0000001 \)</li>
      </ul>
    </ul>
  </div>

  <div class="section">
    <h2>شرح المعادلة:</h2>
    <p>
      هذه المعادلة تقدم حلاً مباشرًا لاستخراج الجذر غير البديهي رقم \( k \) لدالة زيتا في المستوى المركب.
    </p>
    <p>
      تمثل \( \theta \) الزاوية الطيفية الموحدة، وتدخل \( k \) ضمن بنية الجذر من خلال معامل زاوي \( 2\theta k \).
    </p>
    <p>
      الحد \( C_0 \) هو إزاحة ثابتة تضمن التمركز الصحيح للأصفار في الحقل الطيفي.
    </p>
    <p>
      كثيرة الحدود \( \beta_n \left( \frac{\ln k}{\ln \ln k} \right)^n \) تمثل التصحيح الديناميكي اللازم لتعديل الدقة مع تزايد \( k \).
    </p>
    <p>
      كل هذه العناصر مقسومة على التردد الطيفي الموحد \( f = 1 \) لضمان ثبات النطاق.
    </p>
  </div>

</body>
</html>
