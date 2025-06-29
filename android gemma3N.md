# Complete Android Chat App with Gemma 3n Integration

Building a production-ready Android chat application that combines WebView frontend with native Gemma 3n AI inference requires careful architecture planning and implementation. This comprehensive guide provides everything needed to create a fully functional chat app using the latest 2025 Android development practices.

## Project architecture and setup

The application architecture centers on **WebView** for UI presentation while keeping **AI inference entirely native** in Kotlin. This design provides the flexibility of web technologies for the chat interface while maximizing performance through native AI processing.

### Core dependencies and build configuration

The project requires specific dependencies for Gemma 3n integration and WebView communication. Configure your `build.gradle` files with these essential components:

**Project-level build.gradle**:
```gradle
allprojects {
    repositories {
        mavenCentral()
        maven {
            name 'ossrh-snapshot'
            url 'https://oss.sonatype.org/content/repositories/snapshots'
        }
        maven { url 'https://huggingface.co' }
    }
}
```

**App-level build.gradle**:
```gradle
android {
    compileSdk 35
    buildToolsVersion "35.0.0"
    
    defaultConfig {
        applicationId "com.yourpackage.gemmachat"
        minSdk 26  // Android 8.0 minimum for Gemma 3n
        targetSdk 35
        multiDexEnabled true
        
        ndk {
            abiFilters 'arm64-v8a', 'armeabi-v7a'
        }
    }
    
    buildTypes {
        release {
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'),
                         'proguard-rules.pro'
        }
    }
    
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
    
    packagingOptions {
        pickFirst '**/libtensorflowlite_jni.so'
        pickFirst '**/libc++_shared.so'
    }
}

dependencies {
    // Core Android dependencies
    implementation 'androidx.appcompat:appcompat:1.6.1'
    implementation 'androidx.constraintlayout:constraintlayout:2.1.4'
    implementation 'com.google.android.material:material:1.11.0'
    implementation 'androidx.lifecycle:lifecycle-viewmodel:2.7.0'
    implementation 'androidx.lifecycle:lifecycle-livedata:2.7.0'
    
    // Gemma 3n and AI inference
    implementation 'com.google.ai.edge.litert:litert:0.0.0-SNAPSHOT'
    implementation 'com.google.ai.edge.litert:litert-gpu:0.0.0-SNAPSHOT'
    implementation 'com.google.mediapipe:tasks-genai:0.10.14'
    
    // JSON processing and HTTP client
    implementation 'com.google.code.gson:gson:2.10.1'
    implementation 'com.squareup.okhttp3:okhttp:4.12.0'
    
    // Coroutines for async processing
    implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-android:1.7.3'
}
```

### Android manifest configuration

The manifest must include specific permissions and configurations for AI model access and WebView functionality:

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.yourpackage.gemmachat">

    <!-- Essential permissions -->
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"
                     android:maxSdkVersion="28" />
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />

    <!-- Hardware features for potential multimodal use -->
    <uses-feature android:name="android.hardware.camera" android:required="false" />
    <uses-feature android:name="android.hardware.microphone" android:required="false" />

    <application
        android:name=".ChatApplication"
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:theme="@style/AppTheme"
        android:networkSecurityConfig="@xml/network_security_config">

        <!-- Native library declarations for Android 12+ -->
        <uses-native-library 
            android:name="libOpenCL.so" 
            android:required="false" />
        <uses-native-library 
            android:name="libOpenCL-pixel.so" 
            android:required="false" />

        <!-- File provider for model downloads -->
        <provider
            android:name="androidx.core.content.FileProvider"
            android:authorities="${applicationId}.fileprovider"
            android:exported="false"
            android:grantUriPermissions="true">
            <meta-data
                android:name="android.support.FILE_PROVIDER_PATHS"
                android:resource="@xml/file_paths" />
        </provider>

        <activity android:name=".MainActivity"
                  android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>
</manifest>
```

**Network Security Configuration** (`res/xml/network_security_config.xml`):
```xml
<network-security-config>
    <domain-config cleartextTrafficPermitted="true">
        <domain includeSubdomains="true">localhost</domain>
        <domain includeSubdomains="true">192.168.1.1</domain>
    </domain-config>
</network-security-config>
```

**File Paths Configuration** (`res/xml/file_paths.xml`):
```xml
<paths>
    <files-path name="models" path="models/" />
    <cache-path name="cache" path="." />
</paths>
```

## XML layout implementation

The UI uses traditional XML layouts with Material Design components, providing a clean foundation for the WebView-based chat interface.

### Main activity layout

**res/layout/activity_main.xml**:
```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout 
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <!-- Progress indicator for model loading -->
    <com.google.android.material.progressindicator.LinearProgressIndicator
        android:id="@+id/loadingIndicator"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:visibility="visible"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent" />

    <!-- Status text for loading feedback -->
    <TextView
        android:id="@+id/statusText"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:text="@string/loading_model"
        android:textAlignment="center"
        android:padding="16dp"
        android:textSize="16sp"
        app:layout_constraintTop_toBottomOf="@+id/loadingIndicator"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent" />

    <!-- WebView container for chat interface -->
    <FrameLayout
        android:id="@+id/webviewContainer"
        android:layout_width="0dp"
        android:layout_height="0dp"
        android:visibility="gone"
        app:layout_constraintTop_toBottomOf="@+id/statusText"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent">

        <WebView
            android:id="@+id/chatWebView"
            android:layout_width="match_parent"
            android:layout_height="match_parent" />

    </FrameLayout>

    <!-- Error state layout -->
    <LinearLayout
        android:id="@+id/errorLayout"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        android:padding="24dp"
        android:visibility="gone"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent">

        <ImageView
            android:layout_width="64dp"
            android:layout_height="64dp"
            android:layout_gravity="center_horizontal"
            android:src="@drawable/ic_error"
            android:tint="?attr/colorError" />

        <TextView
            android:id="@+id/errorMessage"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="center_horizontal"
            android:layout_marginTop="16dp"
            android:text="@string/model_load_error"
            android:textAlignment="center"
            android:textSize="16sp" />

        <com.google.android.material.button.MaterialButton
            android:id="@+id/retryButton"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="center_horizontal"
            android:layout_marginTop="16dp"
            android:text="@string/retry" />

    </LinearLayout>

</androidx.constraintlayout.widget.ConstraintLayout>
```

### Styles and themes

**res/values/styles.xml**:
```xml
<resources>
    <style name="AppTheme" parent="Theme.Material3.DayNight">
        <item name="colorPrimary">@color/gemma_primary</item>
        <item name="colorSecondary">@color/gemma_secondary</item>
        <item name="colorSurface">@color/gemma_surface</item>
        <item name="android:windowBackground">@color/gemma_background</item>
        <item name="android:statusBarColor">@color/gemma_primary_variant</item>
    </style>
    
    <style name="LoadingTextStyle">
        <item name="android:textSize">16sp</item>
        <item name="android:textColor">?attr/colorOnSurface</item>
        <item name="android:fontFamily">sans-serif-medium</item>
    </style>
</resources>
```

**res/values/colors.xml**:
```xml
<resources>
    <color name="gemma_primary">#2196F3</color>
    <color name="gemma_primary_variant">#1976D2</color>
    <color name="gemma_secondary">#FF9800</color>
    <color name="gemma_surface">#FFFFFF</color>
    <color name="gemma_background">#FAFAFA</color>
    <color name="gemma_error">#F44336</color>
</resources>
```

## Native Gemma 3n integration

The core AI functionality runs entirely in native Kotlin code, providing optimal performance and resource management.

### Model manager implementation

```kotlin
import com.google.mediapipe.tasks.genai.llminference.LlmInference
import kotlinx.coroutines.*
import android.util.Log
import java.io.File

class GemmaModelManager(private val context: Context) {
    private var llmInference: LlmInference? = null
    private var isModelLoaded = false
    
    // Model configuration
    private val MODEL_PATH = "gemma3n-e2b-it.task"
    private val MAX_TOKENS = 2048
    private val TEMPERATURE = 0.7f
    
    suspend fun initializeModel(): Boolean = withContext(Dispatchers.IO) {
        try {
            Log.d("GemmaModel", "Starting model initialization...")
            
            // Check if model file exists in assets
            val modelFile = File(context.filesDir, MODEL_PATH)
            if (!modelFile.exists()) {
                copyModelFromAssets(modelFile)
            }
            
            // Configure LLM inference options
            val options = LlmInference.LlmInferenceOptions.builder()
                .setModelPath(modelFile.absolutePath)
                .setMaxTokens(MAX_TOKENS)
                .setRandomSeed(12345)
                .setTopK(40)
                .setTemperature(TEMPERATURE)
                .build()
            
            // Initialize the model
            llmInference = LlmInference.createFromOptions(context, options)
            isModelLoaded = true
            
            Log.d("GemmaModel", "Model loaded successfully")
            true
        } catch (e: Exception) {
            Log.e("GemmaModel", "Failed to initialize model", e)
            isModelLoaded = false
            false
        }
    }
    
    private suspend fun copyModelFromAssets(destination: File) = withContext(Dispatchers.IO) {
        try {
            context.assets.open(MODEL_PATH).use { input ->
                destination.outputStream().use { output ->
                    input.copyTo(output)
                }
            }
            Log.d("GemmaModel", "Model copied from assets")
        } catch (e: Exception) {
            Log.e("GemmaModel", "Failed to copy model from assets", e)
            throw e
        }
    }
    
    suspend fun generateResponse(prompt: String): Flow<String> = flow {
        if (!isModelLoaded || llmInference == null) {
            throw IllegalStateException("Model not loaded")
        }
        
        try {
            withContext(Dispatchers.IO) {
                llmInference?.generateResponseAsync(prompt)?.let { result ->
                    emit(result.responseText())
                }
            }
        } catch (e: Exception) {
            Log.e("GemmaModel", "Error generating response", e)
            throw e
        }
    }.flowOn(Dispatchers.IO)
    
    fun cleanup() {
        llmInference?.close()
        llmInference = null
        isModelLoaded = false
    }
    
    fun isReady(): Boolean = isModelLoaded
}
```

### Chat service implementation

```kotlin
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.flow
import org.json.JSONObject
import java.util.*

data class ChatMessage(
    val id: String = UUID.randomUUID().toString(),
    val content: String,
    val isUser: Boolean,
    val timestamp: Long = System.currentTimeMillis()
)

class ChatService(
    private val modelManager: GemmaModelManager,
    private val scope: CoroutineScope
) {
    private val conversationHistory = mutableListOf<ChatMessage>()
    private val MAX_HISTORY_SIZE = 20
    
    fun sendMessage(userMessage: String): Flow<ChatMessage> = flow {
        // Add user message to history
        val userMsg = ChatMessage(content = userMessage, isUser = true)
        conversationHistory.add(userMsg)
        
        // Prepare context from conversation history
        val context = buildConversationContext()
        
        try {
            modelManager.generateResponse(context).collect { response ->
                val assistantMsg = ChatMessage(content = response, isUser = false)
                conversationHistory.add(assistantMsg)
                
                // Trim history if too long
                if (conversationHistory.size > MAX_HISTORY_SIZE) {
                    conversationHistory.removeAt(0)
                }
                
                emit(assistantMsg)
            }
        } catch (e: Exception) {
            val errorMsg = ChatMessage(
                content = "I'm having trouble processing that. Please try again.",
                isUser = false
            )
            emit(errorMsg)
            Log.e("ChatService", "Error in chat processing", e)
        }
    }
    
    private fun buildConversationContext(): String {
        val systemPrompt = """You are a helpful AI assistant. Provide clear, concise, and helpful responses. 
            |Keep your responses conversational and engaging.""".trimMargin()
        
        val context = StringBuilder(systemPrompt)
        context.append("\n\nConversation:\n")
        
        conversationHistory.takeLast(10).forEach { message ->
            val role = if (message.isUser) "User" else "Assistant"
            context.append("$role: ${message.content}\n")
        }
        
        context.append("Assistant: ")
        return context.toString()
    }
    
    fun getConversationHistory(): List<ChatMessage> = conversationHistory.toList()
    
    fun clearHistory() {
        conversationHistory.clear()
    }
}
```

## WebView configuration and JavaScript bridge

The WebView serves as the frontend interface while communicating with native Kotlin code through a secure JavaScript bridge.

### Secure WebView setup

```kotlin
import android.webkit.*
import android.net.Uri
import android.os.Build
import org.json.JSONObject

class ChatActivity : AppCompatActivity() {
    private lateinit var webView: WebView
    private lateinit var modelManager: GemmaModelManager
    private lateinit var chatService: ChatService
    private lateinit var javascriptBridge: JavaScriptBridge
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        
        initializeComponents()
        setupWebView()
        initializeModel()
    }
    
    private fun initializeComponents() {
        webView = findViewById(R.id.chatWebView)
        modelManager = GemmaModelManager(this)
        chatService = ChatService(modelManager, lifecycleScope)
        javascriptBridge = JavaScriptBridge(chatService, lifecycleScope)
    }
    
    private fun setupWebView() {
        configureWebViewSecurity()
        setupJavaScriptBridge()
        loadChatInterface()
    }
    
    private fun configureWebViewSecurity() {
        val webSettings = webView.settings
        
        // Enable JavaScript for bridge communication
        webSettings.javaScriptEnabled = true
        
        // Security configurations
        webSettings.allowFileAccess = false
        webSettings.allowContentAccess = false
        webSettings.allowFileAccessFromFileURLs = false
        webSettings.allowUniversalAccessFromFileURLs = false
        webSettings.setGeolocationEnabled(false)
        webSettings.setSaveFormData(false)
        webSettings.setSavePassword(false)
        webSettings.mixedContentMode = WebSettings.MIXED_CONTENT_NEVER_ALLOW
        
        // Performance optimizations
        webSettings.cacheMode = WebSettings.LOAD_DEFAULT
        webSettings.setAppCacheEnabled(true)
        webSettings.domStorageEnabled = true
        webSettings.useWideViewPort = true
        webSettings.loadWithOverviewMode = true
        
        // Disable zoom controls
        webSettings.setSupportZoom(false)
        webSettings.builtInZoomControls = false
        webSettings.displayZoomControls = false
        
        // Set up WebView clients
        webView.webViewClient = SecureChatWebViewClient()
        webView.webChromeClient = ChatWebChromeClient()
        
        // Enable debugging in debug builds
        if (BuildConfig.DEBUG) {
            WebView.setWebContentsDebuggingEnabled(true)
        }
    }
    
    private fun setupJavaScriptBridge() {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
            setupModernBridge()
        } else {
            setupLegacyBridge()
        }
    }
    
    @RequiresApi(Build.VERSION_CODES.M)
    private fun setupModernBridge() {
        webView.webViewClient = object : WebViewClient() {
            override fun onPageFinished(view: WebView?, url: String?) {
                super.onPageFinished(view, url)
                initializeMessageChannel()
            }
        }
    }
    
    @RequiresApi(Build.VERSION_CODES.M)
    private fun initializeMessageChannel() {
        val channel = webView.createWebMessageChannel()
        val ourPort = channel[0]
        val webPort = channel[1]
        
        // Set up message handler
        ourPort.setWebMessageCallback(object : WebMessagePort.WebMessageCallback() {
            override fun onMessage(port: WebMessagePort, message: WebMessage) {
                handleWebMessage(message.data)
            }
        })
        
        // Send port to web page
        val initMessage = WebMessage("init", arrayOf(webPort))
        webView.postWebMessage(initMessage, Uri.parse("*"))
        
        javascriptBridge.setMessagePort(ourPort)
    }
    
    private fun setupLegacyBridge() {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.JELLY_BEAN_MR1) {
            webView.addJavaScriptInterface(
                LegacyJavaScriptInterface(chatService, lifecycleScope),
                "AndroidBridge"
            )
        }
    }
    
    private fun loadChatInterface() {
        val chatHtml = loadChatHtmlFromAssets()
        webView.loadDataWithBaseURL(
            "file:///android_asset/",
            chatHtml,
            "text/html",
            "UTF-8",
            null
        )
    }
    
    private fun loadChatHtmlFromAssets(): String {
        return try {
            assets.open("chat.html").bufferedReader().use { it.readText() }
        } catch (e: Exception) {
            Log.e("ChatActivity", "Failed to load chat HTML", e)
            getDefaultChatHtml()
        }
    }
    
    private fun initializeModel() {
        showLoadingState("Initializing AI model...")
        
        lifecycleScope.launch {
            try {
                val success = modelManager.initializeModel()
                if (success) {
                    showChatInterface()
                } else {
                    showErrorState("Failed to load AI model")
                }
            } catch (e: Exception) {
                showErrorState("Error initializing model: ${e.message}")
            }
        }
    }
    
    private fun showLoadingState(message: String) {
        findViewById<TextView>(R.id.statusText).text = message
        findViewById<View>(R.id.loadingIndicator).visibility = View.VISIBLE
        findViewById<View>(R.id.webviewContainer).visibility = View.GONE
        findViewById<View>(R.id.errorLayout).visibility = View.GONE
    }
    
    private fun showChatInterface() {
        findViewById<View>(R.id.loadingIndicator).visibility = View.GONE
        findViewById<View>(R.id.webviewContainer).visibility = View.VISIBLE
        findViewById<View>(R.id.errorLayout).visibility = View.GONE
    }
    
    private fun showErrorState(message: String) {
        findViewById<TextView>(R.id.errorMessage).text = message
        findViewById<View>(R.id.loadingIndicator).visibility = View.GONE
        findViewById<View>(R.id.webviewContainer).visibility = View.GONE
        findViewById<View>(R.id.errorLayout).visibility = View.VISIBLE
        
        findViewById<View>(R.id.retryButton).setOnClickListener {
            initializeModel()
        }
    }
}
```

### JavaScript bridge implementation

```kotlin
import android.webkit.WebMessagePort
import android.webkit.WebMessage
import kotlinx.coroutines.CoroutineScope
import kotlinx.coroutines.launch
import org.json.JSONObject

class JavaScriptBridge(
    private val chatService: ChatService,
    private val scope: CoroutineScope
) {
    private var messagePort: WebMessagePort? = null
    
    fun setMessagePort(port: WebMessagePort) {
        messagePort = port
    }
    
    fun handleWebMessage(data: String?) {
        try {
            val json = JSONObject(data ?: "")
            val action = json.getString("action")
            val messageId = json.optString("id", "")
            
            when (action) {
                "sendMessage" -> {
                    val message = json.getString("message")
                    handleSendMessage(message, messageId)
                }
                "clearHistory" -> {
                    chatService.clearHistory()
                    sendResponse(messageId, JSONObject().put("success", true))
                }
                "getHistory" -> {
                    val history = chatService.getConversationHistory()
                    val historyJson = JSONObject().put("messages", history.map { msg ->
                        JSONObject().apply {
                            put("id", msg.id)
                            put("content", msg.content)
                            put("isUser", msg.isUser)
                            put("timestamp", msg.timestamp)
                        }
                    })
                    sendResponse(messageId, historyJson)
                }
            }
        } catch (e: Exception) {
            Log.e("JavaScriptBridge", "Error handling web message", e)
            sendError("Invalid message format")
        }
    }
    
    private fun handleSendMessage(message: String, messageId: String) {
        scope.launch {
            try {
                chatService.sendMessage(message).collect { response ->
                    val responseJson = JSONObject().apply {
                        put("type", "messageResponse")
                        put("id", response.id)
                        put("content", response.content)
                        put("isUser", response.isUser)
                        put("timestamp", response.timestamp)
                        put("messageId", messageId)
                    }
                    sendToWeb(responseJson.toString())
                }
            } catch (e: Exception) {
                sendError("Error processing message: ${e.message}")
            }
        }
    }
    
    private fun sendResponse(messageId: String, data: JSONObject) {
        val response = JSONObject().apply {
            put("type", "response")
            put("id", messageId)
            put("data", data)
        }
        sendToWeb(response.toString())
    }
    
    private fun sendError(error: String) {
        val errorJson = JSONObject().apply {
            put("type", "error")
            put("message", error)
        }
        sendToWeb(errorJson.toString())
    }
    
    private fun sendToWeb(message: String) {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
            messagePort?.postMessage(WebMessage(message))
        }
    }
}

// Legacy JavaScript interface for older Android versions
class LegacyJavaScriptInterface(
    private val chatService: ChatService,
    private val scope: CoroutineScope
) {
    @JavascriptInterface
    fun sendMessage(message: String): String {
        var result = ""
        scope.launch {
            try {
                chatService.sendMessage(message).collect { response ->
                    result = JSONObject().apply {
                        put("success", true)
                        put("response", response.content)
                        put("id", response.id)
                    }.toString()
                }
            } catch (e: Exception) {
                result = JSONObject().apply {
                    put("success", false)
                    put("error", e.message)
                }.toString()
            }
        }
        return result
    }
    
    @JavascriptInterface
    fun clearHistory(): String {
        return try {
            chatService.clearHistory()
            JSONObject().put("success", true).toString()
        } catch (e: Exception) {
            JSONObject().put("success", false).put("error", e.message).toString()
        }
    }
}
```

## Web frontend implementation

The chat interface uses modern HTML5, CSS3, and JavaScript while maintaining compatibility with WebView constraints.

### Chat HTML interface

**assets/chat.html**:
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Gemma Chat</title>
    <style>
        :root {
            --primary-color: #2196F3;
            --primary-dark: #1976D2;
            --secondary-color: #FF9800;
            --background-color: #FAFAFA;
            --surface-color: #FFFFFF;
            --text-primary: #212121;
            --text-secondary: #757575;
            --border-color: #E0E0E0;
            --user-message-bg: #E3F2FD;
            --assistant-message-bg: #F5F5F5;
        }
        
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }
        
        body {
            font-family: 'Roboto', -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
            background-color: var(--background-color);
            height: 100vh;
            display: flex;
            flex-direction: column;
        }
        
        .chat-header {
            background-color: var(--primary-color);
            color: white;
            padding: 16px;
            display: flex;
            align-items: center;
            box-shadow: 0 2px 4px rgba(0,0,0,0.1);
        }
        
        .chat-title {
            font-size: 20px;
            font-weight: 500;
        }
        
        .chat-container {
            flex: 1;
            display: flex;
            flex-direction: column;
            min-height: 0;
        }
        
        .messages-container {
            flex: 1;
            overflow-y: auto;
            padding: 16px;
            scroll-behavior: smooth;
        }
        
        .message {
            margin-bottom: 16px;
            animation: fadeInUp 0.3s ease-out;
        }
        
        .message-content {
            max-width: 80%;
            padding: 12px 16px;
            border-radius: 18px;
            line-height: 1.4;
            word-wrap: break-word;
        }
        
        .user-message {
            display: flex;
            justify-content: flex-end;
        }
        
        .user-message .message-content {
            background-color: var(--user-message-bg);
            color: var(--text-primary);
        }
        
        .assistant-message {
            display: flex;
            justify-content: flex-start;
        }
        
        .assistant-message .message-content {
            background-color: var(--assistant-message-bg);
            color: var(--text-primary);
            border: 1px solid var(--border-color);
        }
        
        .message-time {
            font-size: 12px;
            color: var(--text-secondary);
            margin-top: 4px;
            text-align: right;
        }
        
        .assistant-message .message-time {
            text-align: left;
        }
        
        .input-container {
            padding: 16px;
            background-color: var(--surface-color);
            border-top: 1px solid var(--border-color);
            display: flex;
            align-items: flex-end;
            gap: 12px;
        }
        
        .message-input {
            flex: 1;
            min-height: 24px;
            max-height: 120px;
            padding: 12px 16px;
            border: 1px solid var(--border-color);
            border-radius: 24px;
            resize: none;
            font-size: 16px;
            font-family: inherit;
            outline: none;
            background-color: var(--surface-color);
        }
        
        .message-input:focus {
            border-color: var(--primary-color);
        }
        
        .send-button {
            width: 48px;
            height: 48px;
            border: none;
            border-radius: 50%;
            background-color: var(--primary-color);
            color: white;
            cursor: pointer;
            display: flex;
            align-items: center;
            justify-content: center;
            transition: background-color 0.2s;
        }
        
        .send-button:hover {
            background-color: var(--primary-dark);
        }
        
        .send-button:disabled {
            background-color: var(--border-color);
            cursor: not-allowed;
        }
        
        .typing-indicator {
            display: none;
            padding: 12px 16px;
            background-color: var(--assistant-message-bg);
            border-radius: 18px;
            max-width: 80px;
            margin-bottom: 16px;
        }
        
        .typing-dots {
            display: flex;
            gap: 4px;
        }
        
        .typing-dot {
            width: 8px;
            height: 8px;
            border-radius: 50%;
            background-color: var(--text-secondary);
            animation: typing 1.4s infinite ease-in-out;
        }
        
        .typing-dot:nth-child(2) {
            animation-delay: 0.2s;
        }
        
        .typing-dot:nth-child(3) {
            animation-delay: 0.4s;
        }
        
        @keyframes fadeInUp {
            from {
                opacity: 0;
                transform: translateY(20px);
            }
            to {
                opacity: 1;
                transform: translateY(0);
            }
        }
        
        @keyframes typing {
            0%, 60%, 100% {
                transform: scale(0.8);
                opacity: 0.5;
            }
            30% {
                transform: scale(1);
                opacity: 1;
            }
        }
        
        .error-message {
            background-color: #ffebee;
            border-left: 4px solid #f44336;
            padding: 12px;
            margin-bottom: 16px;
            border-radius: 4px;
            color: #c62828;
            font-size: 14px;
        }
        
        @media (max-width: 480px) {
            .message-content {
                max-width: 90%;
            }
            
            .input-container {
                padding: 12px;
            }
        }
    </style>
</head>
<body>
    <div class="chat-header">
        <div class="chat-title">Gemma Chat Assistant</div>
    </div>
    
    <div class="chat-container">
        <div class="messages-container" id="messagesContainer">
            <div class="message assistant-message">
                <div class="message-content">
                    Hello! I'm your AI assistant powered by Gemma 3n. How can I help you today?
                    <div class="message-time" id="welcomeTime"></div>
                </div>
            </div>
        </div>
        
        <div class="typing-indicator" id="typingIndicator">
            <div class="typing-dots">
                <div class="typing-dot"></div>
                <div class="typing-dot"></div>
                <div class="typing-dot"></div>
            </div>
        </div>
        
        <div class="input-container">
            <textarea 
                class="message-input" 
                id="messageInput" 
                placeholder="Type your message..."
                rows="1"></textarea>
            <button class="send-button" id="sendButton">
                <svg width="24" height="24" viewBox="0 0 24 24" fill="currentColor">
                    <path d="M2.01 21L23 12 2.01 3 2 10l15 2-15 2z"/>
                </svg>
            </button>
        </div>
    </div>

    <script>
        class ChatInterface {
            constructor() {
                this.messagesContainer = document.getElementById('messagesContainer');
                this.messageInput = document.getElementById('messageInput');
                this.sendButton = document.getElementById('sendButton');
                this.typingIndicator = document.getElementById('typingIndicator');
                this.nativePort = null;
                
                this.initializeInterface();
                this.setupEventListeners();
            }
            
            initializeInterface() {
                // Set welcome message time
                document.getElementById('welcomeTime').textContent = this.formatTime(new Date());
                
                // Initialize native bridge
                this.initializeBridge();
            }
            
            initializeBridge() {
                // Modern WebMessagePort approach
                window.addEventListener('message', (event) => {
                    if (event.data === 'init' && event.ports && event.ports[0]) {
                        this.nativePort = event.ports[0];
                        this.nativePort.onmessage = (event) => {
                            this.handleNativeMessage(JSON.parse(event.data));
                        };
                        this.nativePort.start();
                        console.log('Bridge initialized with WebMessagePort');
                    }
                });
                
                // Fallback to legacy interface
                if (window.AndroidBridge) {
                    console.log('Using legacy JavaScript interface');
                }
            }
            
            setupEventListeners() {
                this.sendButton.addEventListener('click', () => {
                    this.sendMessage();
                });
                
                this.messageInput.addEventListener('keypress', (e) => {
                    if (e.key === 'Enter' && !e.shiftKey) {
                        e.preventDefault();
                        this.sendMessage();
                    }
                });
                
                this.messageInput.addEventListener('input', () => {
                    this.adjustTextareaHeight();
                });
            }
            
            adjustTextareaHeight() {
                this.messageInput.style.height = 'auto';
                this.messageInput.style.height = this.messageInput.scrollHeight + 'px';
            }
            
            sendMessage() {
                const message = this.messageInput.value.trim();
                if (!message) return;
                
                // Display user message
                this.addMessage(message, true);
                
                // Clear input
                this.messageInput.value = '';
                this.adjustTextareaHeight();
                
                // Disable send button and show typing indicator
                this.setSendingState(true);
                
                // Send to native
                this.sendToNative('sendMessage', { message: message });
            }
            
            sendToNative(action, data = {}) {
                const message = {
                    action: action,
                    id: this.generateId(),
                    ...data
                };
                
                if (this.nativePort) {
                    // Modern approach
                    this.nativePort.postMessage(JSON.stringify(message));
                } else if (window.AndroidBridge) {
                    // Legacy approach
                    const result = window.AndroidBridge.sendMessage(data.message);
                    try {
                        const response = JSON.parse(result);
                        if (response.success) {
                            this.addMessage(response.response, false);
                        } else {
                            this.showError(response.error);
                        }
                    } catch (e) {
                        this.showError('Failed to parse response');
                    }
                    this.setSendingState(false);
                }
            }
            
            handleNativeMessage(message) {
                switch (message.type) {
                    case 'messageResponse':
                        this.addMessage(message.content, false);
                        this.setSendingState(false);
                        break;
                    case 'error':
                        this.showError(message.message);
                        this.setSendingState(false);
                        break;
                    case 'response':
                        // Handle other response types
                        break;
                }
            }
            
            addMessage(content, isUser) {
                const messageDiv = document.createElement('div');
                messageDiv.className = `message ${isUser ? 'user-message' : 'assistant-message'}`;
                
                const contentDiv = document.createElement('div');
                contentDiv.className = 'message-content';
                contentDiv.textContent = content;
                
                const timeDiv = document.createElement('div');
                timeDiv.className = 'message-time';
                timeDiv.textContent = this.formatTime(new Date());
                
                contentDiv.appendChild(timeDiv);
                messageDiv.appendChild(contentDiv);
                
                this.messagesContainer.appendChild(messageDiv);
                this.scrollToBottom();
            }
            
            setSendingState(isSending) {
                this.sendButton.disabled = isSending;
                this.messageInput.disabled = isSending;
                this.typingIndicator.style.display = isSending ? 'block' : 'none';
                if (isSending) {
                    this.scrollToBottom();
                }
            }
            
            showError(message) {
                const errorDiv = document.createElement('div');
                errorDiv.className = 'error-message';
                errorDiv.textContent = message;
                this.messagesContainer.appendChild(errorDiv);
                this.scrollToBottom();
            }
            
            formatTime(date) {
                return date.toLocaleTimeString([], { 
                    hour: '2-digit', 
                    minute: '2-digit' 
                });
            }
            
            generateId() {
                return Math.random().toString(36).substr(2, 9);
            }
            
            scrollToBottom() {
                setTimeout(() => {
                    this.messagesContainer.scrollTop = this.messagesContainer.scrollHeight;
                }, 100);
            }
        }
        
        // Initialize chat interface when DOM is loaded
        document.addEventListener('DOMContentLoaded', () => {
            new ChatInterface();
        });
    </script>
</body>
</html>
```

## Error handling and performance optimization

Comprehensive error handling ensures the application remains stable and provides meaningful feedback to users.

### Error handling implementation

```kotlin
sealed class ChatError : Exception() {
    object ModelNotLoaded : ChatError()
    object NetworkError : ChatError()
    object InsufficientMemory : ChatError()
    data class InferenceError(val details: String) : ChatError()
    data class BridgeError(val message: String) : ChatError()
}

class ErrorHandler {
    fun handleError(error: Throwable): String {
        return when (error) {
            is ChatError.ModelNotLoaded -> 
                "AI model is not ready. Please wait a moment and try again."
            is ChatError.NetworkError -> 
                "Network connection required for initial setup."
            is ChatError.InsufficientMemory -> 
                "Device memory is low. Try closing other apps."
            is ChatError.InferenceError -> 
                "AI processing error: ${error.details}"
            is ChatError.BridgeError -> 
                "Communication error: ${error.message}"
            else -> 
                "An unexpected error occurred. Please try again."
        }
    }
}
```

### Performance monitoring

```kotlin
class PerformanceMonitor {
    private var startTime: Long = 0
    
    fun startInference() {
        startTime = System.currentTimeMillis()
    }
    
    fun endInference(): Long {
        return System.currentTimeMillis() - startTime
    }
    
    fun logPerformanceMetrics(responseTime: Long, memoryUsage: Long) {
        Log.d("Performance", "Response time: ${responseTime}ms, Memory: ${memoryUsage}MB")
        
        // Send to analytics if needed
        if (responseTime > 5000) {
            Log.w("Performance", "Slow response detected: ${responseTime}ms")
        }
    }
}
```

## Testing and debugging approaches  

Comprehensive testing ensures reliability across different device configurations and usage patterns.

### Unit testing setup

```kotlin
@RunWith(AndroidJUnit4::class)
class ChatServiceTest {
    
    @Mock
    private lateinit var mockModelManager: GemmaModelManager
    
    private lateinit var chatService: ChatService
    private val testScope = TestCoroutineScope()
    
    @Before
    fun setup() {
        MockitoAnnotations.openMocks(this)
        chatService = ChatService(mockModelManager, testScope)
    }
    
    @Test
    fun testSendMessage_Success() = testScope.runBlockingTest {
        // Arrange
        val testMessage = "Hello, AI!"
        val expectedResponse = "Hello! How can I help you?"
        
        `when`(mockModelManager.generateResponse(any())).thenReturn(
            flowOf(expectedResponse)
        )
        
        // Act
        val responses = mutableListOf<ChatMessage>()
        chatService.sendMessage(testMessage).collect { response ->
            responses.add(response)
        }
        
        // Assert
        assertEquals(1, responses.size)
        assertEquals(expectedResponse, responses[0].content)
        assertFalse(responses[0].isUser)
    }
    
    @Test
    fun testConversationHistory_Trimming() {
        // Test that conversation history is properly trimmed
        repeat(25) { index ->
            runBlocking {
                chatService.sendMessage("Message $index").collect { }
            }
        }
        
        val history = chatService.getConversationHistory()
        assertTrue("History should be trimmed", history.size <= 20)
    }
}
```

### Integration testing

```kotlin
@RunWith(AndroidJUnit4::class)
@LargeTest
class ChatIntegrationTest {
    
    @get:Rule
    val activityRule = ActivityTestRule(ChatActivity::class.java)
    
    @Test
    fun testFullChatFlow() {
        // Wait for model to load
        onView(withId(R.id.loadingIndicator)).check(matches(isDisplayed()))
        
        // Wait for chat interface to appear (may take time for model loading)
        onView(withId(R.id.chatWebView))
            .perform(waitForWebViewToLoad())
            .check(matches(isDisplayed()))
        
        // Test sending a message through WebView
        onView(withId(R.id.chatWebView))
            .perform(executeJavaScript("""
                document.getElementById('messageInput').value = 'Hello AI';
                document.getElementById('sendButton').click();
            """))
        
        // Verify response appears (wait for AI processing)
        Thread.sleep(5000) // Allow time for AI inference
        
        onView(withId(R.id.chatWebView))
            .check(webViewHasContent("assistant-message"))
    }
}
```

### Debugging utilities

```kotlin
class DebugUtils {
    companion object {
        fun enableWebViewDebugging() {
            if (BuildConfig.DEBUG && Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {
                WebView.setWebContentsDebuggingEnabled(true)
            }
        }
        
        fun logModelInfo(modelManager: GemmaModelManager) {
            Log.d("Debug", "Model loaded: ${modelManager.isReady()}")
            Log.d("Debug", "Available memory: ${getAvailableMemory()}")
        }
        
        private fun getAvailableMemory(): String {
            val runtime = Runtime.getRuntime()
            val maxMemory = runtime.maxMemory() / (1024 * 1024)
            val totalMemory = runtime.totalMemory() / (1024 * 1024)
            val freeMemory = runtime.freeMemory() / (1024 * 1024)
            return "Max: ${maxMemory}MB, Total: ${totalMemory}MB, Free: ${freeMemory}MB"
        }
    }
}
```

This comprehensive implementation guide provides a complete, production-ready Android chat application with native Gemma 3n integration. The architecture balances performance, security, and maintainability while leveraging the latest Android development practices and AI capabilities available in 2025.

The key advantages of this approach include **native AI processing** for optimal performance, **secure WebView communication** through modern JavaScript bridges, **robust error handling** for production reliability, and **comprehensive testing strategies** for quality assurance.