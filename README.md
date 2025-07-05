<!DOCTYPE html><html lang="ar">
<head>
  <meta charset="UTF-8">
  <title>معادلة علاء المنطقية الطيفية (ASF)</title>
  <style>
    body {
      font-family: 'Arial', sans-serif;
      background: #f4f4f4;
      padding: 20px;
      direction: rtl;
    }
    .container {
      background: #fff;
      border-radius: 10px;
      padding: 30px;
      box-shadow: 0 0 10px rgba(0,0,0,0.1);
      line-height: 1.8;
    }
    h1 {
      color: #2c3e50;
      text-align: center;
    }
    .equation {
      background: #f0f0f0;
      border: 1px solid #ddd;
      padding: 10px;
      font-size: 18px;
      direction: ltr;
      text-align: center;
      margin: 20px 0;
    }
    .footer {
      margin-top: 40px;
      text-align: center;
      color: #777;
    }
  </style>
</head>
<body>
  <div class="container">
    <h1>📘 المعادلة المنطقية الطيفية لـ علاء (ASF)</h1><p>
  هذه المعادلة تمثل نموذجًا طيفيًا مفلترًا منطقيًا لحالات SAT، بحيث تمنع الحالات الخاطئة من الوصول إلى الرنين الطيفي.
</p>

<div class="equation">
  Im(sₖ) = Im(s₀) + r · (tₖ − Im(s₀)) · sin(θ) · 𝟙<sub>SAT(x,y,z,...)</sub>
</div>

<p><strong>تفصيل الرموز:</strong></p>
<ul>
  <li><strong>Im(s₀):</strong> الطور الأساسي للحالة قبل الانزياح (يُحسب من A عبر معادلة علاء)</li>
  <li><strong>tₖ:</strong> قيمة الرنين المستهدفة</li>
  <li><strong>r:</strong> نسبة التفاعل (عادةً بين 0 و 1)</li>
  <li><strong>θ:</strong> زاوية الحالة المنطقية المستخرجة من ترميزها</li>
  <li><strong>𝟙<sub>SAT(x,y,z,...)</sub>:</strong> دالة تحقق منطقي (تعطي 1 إذا تحققت الجملة، و0 إن لم تتحقق)</li>
</ul>

<p>
  هذا النموذج يمنع أي حالة من الوصول إلى الرنين ما لم تكن منطقية صحيحة، وهو ما يحقق أعلى درجات الأمان الطيفي في تحليل حالات SAT.
</p>

<div class="footer">
  المُعادلة صاغها: <strong>ALAA Sheikh Albasatneh</strong><br>
  الجنسية: <strong>سوري</strong><br>
  التاريخ: <strong>5 تموز 2025</strong>
</div>

  </div>
</body>
</html>
