<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>صانع الأوامر الاحترافية | Prompt Enhancer</title>
    <style>
        :root {
            --primary: #6200ea;
            --background: #f4f5f7;
            --surface: #ffffff;
            --text: #333333;
        }
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: var(--background);
            color: var(--text);
            margin: 0;
            padding: 20px;
            display: flex;
            justify-content: center;
        }
        .container {
            background-color: var(--surface);
            padding: 30px;
            border-radius: 12px;
            box-shadow: 0 4px 12px rgba(0,0,0,0.1);
            width: 100%;
            max-width: 600px;
        }
        h1 {
            color: var(--primary);
            text-align: center;
            font-size: 24px;
        }
        label {
            font-weight: bold;
            display: block;
            margin-top: 15px;
            margin-bottom: 5px;
        }
        input, textarea {
            width: 100%;
            padding: 12px;
            border: 1px solid #ccc;
            border-radius: 8px;
            box-sizing: border-box;
            font-family: inherit;
        }
        textarea {
            resize: vertical;
            min-height: 100px;
        }
        .buttons-container {
            display: flex;
            gap: 10px;
            margin-top: 15px;
        }
        button {
            flex: 1;
            padding: 12px;
            background-color: var(--primary);
            color: white;
            border: none;
            border-radius: 8px;
            cursor: pointer;
            font-size: 16px;
            font-weight: bold;
            transition: background 0.3s;
        }
        button:hover {
            background-color: #4500b5;
        }
        button:disabled {
            background-color: #aaa;
            cursor: not-allowed;
        }
        .copy-btn {
            background-color: #28a745;
            margin-top: 10px;
        }
        .copy-btn:hover {
            background-color: #218838;
        }
        #loading {
            text-align: center;
            color: var(--primary);
            font-weight: bold;
            display: none;
            margin-top: 15px;
        }
    </style>
</head>
<body>

<div class="container">
    <h1>صانع الأوامر الاحترافية 🚀</h1>

    <label for="apiKey">مفتاح API الخاص بـ Gemini (مطلوب للتشغيل):</label>
    <input type="password" id="apiKey" placeholder="ألصق مفتاح API هنا...">

    <label for="userInput">الفكرة البسيطة:</label>
    <textarea id="userInput" placeholder="اكتب فكرتك هنا (مثال: أريد مقال عن التقنية، أو صورة رجل يقرأ في مقهى)..."></textarea>

    <div class="buttons-container">
        <button id="btnText" onclick="generatePrompt('text')">صياغة نص (لـ Gemini)</button>
        <button id="btnImage" onclick="generatePrompt('image')">صياغة صورة (لـ Nano Banana)</button>
    </div>

    <div id="loading">جاري الصياغة... يرجى الانتظار ⏳</div>

    <label for="outputResult">البرومبت الاحترافي الجاهز:</label>
    <textarea id="outputResult" readonly placeholder="ستظهر النتيجة هنا..."></textarea>
    
    <button class="copy-btn" onclick="copyResult()">نسخ النتيجة 📋</button>
</div>

<script>
    async function generatePrompt(type) {
        const apiKey = document.getElementById('apiKey').value.trim();
        const userInput = document.getElementById('userInput').value.trim();
        const outputResult = document.getElementById('outputResult');
        const loading = document.getElementById('loading');
        const buttons = document.querySelectorAll('.buttons-container button');

        if (!apiKey) {
            alert('الرجاء إدخال مفتاح API الخاص بـ Gemini أولاً.');
            return;
        }
        if (!userInput) {
            alert('الرجاء كتابة فكرة بسيطة في مربع النص.');
            return;
        }

        // إعداد التعليمة المخصصة بناءً على نوع الطلب (نص أو صورة)
        let systemPrompt = "";
        if (type === 'text') {
            systemPrompt = `أنت خبير محترف في هندسة الأوامر. 
المستخدم سيعطيك فكرة بسيطة. قم بصياغة "أمر (Prompt)" دقيق ومفصل لنموذج ذكاء اصطناعي (مثل Gemini) لطلب محتوى نصي أو عصف ذهني.
اطلب أفكاراً متسلسلة، زوايا إبداعية، وهيكلة واضحة.
الفكرة المبدئية هي: ${userInput}`;
        } else {
            systemPrompt = `أنت خبير محترف في هندسة الأوامر الخاصة بالصور. 
المستخدم سيعطيك فكرة بسيطة. قم بصياغة "وصف بصري دقيق (Prompt)" لمولد صور (مثل نانو بنانا) باللغة الإنجليزية لتكون دقيقة.
صف الإضاءة، التكوين، الألوان، والنمط الفني.
إذا تضمن الطلب أكثر من طفل، أضف تعليمة صارمة بدمج الأطفال في صورة واحدة بواقعية تامة.
في نهاية البرومبت، أضف دائماً تعليمة صارمة بدمج العلامة المائية "Nos Gady 0.5" وشعار "mdmjan" بحيث يكون الشعار كأنه مدمجاً داخل التكوين، شفافاً قليلاً، ولا إطارات عليه.
الفكرة المبدئية هي: ${userInput}`;
        }

        // إظهار التحميل وتعطيل الأزرار
        loading.style.display = 'block';
        buttons.forEach(btn => btn.disabled = true);
        outputResult.value = "";

        try {
            const response = await fetch(`https://generativelanguage.googleapis.com/v1beta/models/gemini-1.5-flash:generateContent?key=${apiKey}`, {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify({
                    contents: [{
                        parts: [{ text: systemPrompt }]
                    }]
                })
            });

            if (!response.ok) {
                throw new Error('فشل الاتصال بـ API. تأكد من صحة المفتاح.');
            }

            const data = await response.json();
            const generatedText = data.candidates[0].content.parts[0].text;
            outputResult.value = generatedText;

        } catch (error) {
            outputResult.value = "حدث خطأ: " + error.message;
        } finally {
            // إخفاء التحميل وإعادة تفعيل الأزرار
            loading.style.display = 'none';
            buttons.forEach(btn => btn.disabled = false);
        }
    }

    function copyResult() {
        const outputText = document.getElementById('outputResult');
        if (outputText.value) {
            outputText.select();
            document.execCommand('copy');
            alert('تم نسخ البرومبت بنجاح! 🚀');
        } else {
            alert('لا يوجد شيء لنسخه.');
        }
    }
</script>

</body>
</html>
