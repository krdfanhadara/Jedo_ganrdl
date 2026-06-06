dependencies {
    // ... (الاعتماديات الافتراضية الخاصة بـ Compose و Core)
    
    // MediaPipe LLM Inference
    implementation("com.google.mediapipe:tasks-genai:0.10.11")
    
    // Coroutines (إذا لم تكن مضافة)
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-android:1.7.3")
}
package com.yourpackage.name // استبدل هذا باسم الحزمة الخاصة بك

import android.content.Context
import com.google.mediapipe.tasks.genai.llminference.LlmInference
import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.withContext

class LlmManager(private val context: Context) {
    
    private var llmInference: LlmInference? = null
    
    // مسار ملف نموذج Gemma 2B (تأكد من وجود الملف في هذا المسار على الهاتف)
    private val modelPath = "/data/local/tmp/gemma-2b-it-gpu-int4.bin" 

    fun initializeModel() {
        try {
            val options = LlmInference.LlmInferenceOptions.builder()
                .setModelPath(modelPath)
                .setMaxTokens(1024)
                .build()
            
            llmInference = LlmInference.createFromOptions(context, options)
        } catch (e: Exception) {
            e.printStackTrace()
        }
    }

    suspend fun generateProfessionalPrompt(userInput: String): String {
        return withContext(Dispatchers.IO) {
            val systemPrompt = """
                أنت خبير محترف في هندسة الأوامر (Prompt Engineering). 
                المستخدم سيعطيك فكرة بسيطة. مهمتك هي صياغة أمر (Prompt) احترافي ومفصل:
                
                1. إذا كان الطلب يخص "العصف الذهني أو كتابة المحتوى":
                   - صغ الأمر ليكون موجهاً لنموذج "Gemini".
                   - اطلب أفكاراً متسلسلة، زوايا إبداعية، وهيكلة واضحة.
                   
                2. إذا كان الطلب يخص "توليد الصور":
                   - صغ الأمر كـ وصف بصري دقيق (الإضاءة، التكوين، الألوان، النمط الفني).
                   - عند طلب دمج عناصر أو أشخاص في صورة واحدة، اكتب تعليمة صارمة لجعل الدمج واقعياً تماماً.
                   - عند إضافة العلامة المائية "Nos Gady 0.5" أو الشعار "mdmjan"، أضف تعليمة واضحة بأن يكون الشعار مدمجاً داخل التكوين كجزء من الصورة، شفافاً قليلاً، وبدون أي إطارات خارجية.
                
                الطلب البسيط: $userInput
                البرومبت الاحترافي:
            """.trimIndent()

            try {
                llmInference?.generateResponse(systemPrompt) ?: "حدث خطأ: تأكد من تحميل النموذج على الهاتف."
            } catch (e: Exception) {
                "خطأ في المعالجة: ${e.localizedMessage}"
            }
        }
    }

    fun close() {
        llmInference?.close()
    }
}
package com.yourpackage.name // استبدل هذا باسم الحزمة الخاصة بك

import android.content.ClipData
import android.content.ClipboardManager
import android.content.Context
import android.os.Bundle
import android.widget.Toast
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.compose.foundation.layout.*
import androidx.compose.foundation.rememberScrollState
import androidx.compose.foundation.verticalScroll
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.ui.Modifier
import androidx.compose.ui.platform.LocalContext
import androidx.compose.ui.unit.dp
import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.launch

class MainActivity : ComponentActivity() {
    
    private lateinit var llmManager: LlmManager

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        llmManager = LlmManager(this)
        
        // تشغيل التهيئة في الـ Background لتجنب تجميد الشاشة
        Thread {
            llmManager.initializeModel()
        }.start()

        setContent {
            MaterialTheme {
                Surface(
                    modifier = Modifier.fillMaxSize(),
                    color = MaterialTheme.colorScheme.background
                ) {
                    PromptEnhancerApp(llmManager)
                }
            }
        }
    }

    override fun onDestroy() {
        super.onDestroy()
        llmManager.close()
    }
}

@Composable
fun PromptEnhancerApp(llmManager: LlmManager) {
    val context = LocalContext.current
    var userInput by remember { mutableStateOf("") }
    var resultText by remember { mutableStateOf("سيظهر البرومبت الاحترافي هنا...") }
    var isLoading by remember { mutableStateOf(false) }
    val coroutineScope = rememberCoroutineScope()

    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(16.dp)
            .verticalScroll(rememberScrollState())
    ) {
        Text(
            text = "صانع الأوامر الاحترافية", 
            style = MaterialTheme.typography.headlineMedium,
            color = MaterialTheme.colorScheme.primary
        )
        
        Spacer(modifier = Modifier.height(16.dp))

        OutlinedTextField(
            value = userInput,
            onValueChange = { userInput = it },
            label = { Text("أدخل فكرتك البسيطة (نص أو صورة)") },
            modifier = Modifier
                .fillMaxWidth()
                .height(150.dp)
        )

        Spacer(modifier = Modifier.height(16.dp))

        Button(
            onClick = {
                if (userInput.isNotBlank()) {
                    isLoading = true
                    resultText = "جاري الصياغة محلياً... يرجى الانتظار."
                    coroutineScope.launch {
                        resultText = llmManager.generateProfessionalPrompt(userInput)
                        isLoading = false
                    }
                }
            },
            modifier = Modifier.fillMaxWidth(),
            enabled = !isLoading
        ) {
            Text(if (isLoading) "جاري المعالجة..." else "تحويل إلى برومبت احترافي")
        }

        Spacer(modifier = Modifier.height(24.dp))

        Card(
            modifier = Modifier
                .fillMaxWidth()
                .defaultMinSize(minHeight = 200.dp),
            elevation = CardDefaults.cardElevation(defaultElevation = 4.dp)
        ) {
            Text(
                text = resultText,
                modifier = Modifier.padding(16.dp),
                style = MaterialTheme.typography.bodyLarge
            )
        }

        Spacer(modifier = Modifier.height(16.dp))

        // أزرار النسخ لتسريع سير العمل
        Row(
            modifier = Modifier.fillMaxWidth(),
            horizontalArrangement = Arrangement.SpaceEvenly
        ) {
            OutlinedButton(
                onClick = { copyToClipboard(context, resultText, "تم النسخ لـ Gemini") },
                enabled = resultText.isNotBlank() && !isLoading
            ) {
                Text("نسخ للنصوص (Gemini)")
            }

            OutlinedButton(
                onClick = { copyToClipboard(context, resultText, "تم النسخ لتوليد الصور") },
                enabled = resultText.isNotBlank() && !isLoading
            ) {
                Text("نسخ للصور")
            }
        }
    }
}

// دالة مساعدة لنسخ النص إلى الحافظة
fun copyToClipboard(context: Context, text: String, toastMessage: String) {
    val clipboard = context.getSystemService(Context.CLIPBOARD_SERVICE) as ClipboardManager
    val clip = ClipData.newPlainText("Professional Prompt", text)
    clipboard.setPrimaryClip(clip)
    Toast.makeText(context, toastMessage, Toast.LENGTH_SHORT).show()
}
