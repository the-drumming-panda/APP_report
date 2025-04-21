# 🎙️ SecureEcho - Phishing Detection from Voice Messages

[![Android](https://img.shields.io/badge/platform-Android-brightgreen.svg)]()
[![Built with Kotlin](https://img.shields.io/badge/built%20with-Kotlin-purple)]()
[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)]()

SecureEcho is an intelligent Android application that analyzes voice messages to detect phishing attempts. It seamlessly converts `.mp3` audio files into text and evaluates them for malicious content using a smart phishing detection API.

---

## 🚀 Features

- 🎧 Upload MP3 audio files securely
- 🧠 Speech-to-text conversion via REST API
- 🕵️ Phishing intent detection with confidence scores
- 🌈 Visual representation of phishing risk
- ⚙️ Clean architecture (MVVM) using Retrofit & Kotlin

---

## 🏗️ Project Structure

```
SecureEcho/
├── app/
│   └── src/
│       └── main/
│           ├── java/com/secureecho/
│           │   ├── MainActivity.kt
│           │   ├── api/
│           │   │   ├── ApiService.kt
│           │   │   └── RetrofitClient.kt
│           │   ├── models/
│           │   │   ├── AnalysisResponse.kt
│           │   │   └── TranscriptionResponse.kt
│           │   └── utils/
│           │       └── FileUtil.kt
│           ├── res/
│           │   ├── layout/
│           │   │   ├── activity_main.xml
│           │   │   └── analysis_popup.xml
│           │   └── values/
│           │       ├── colors.xml
│           │       ├── strings.xml
│           │       └── themes.xml
│           └── AndroidManifest.xml
└── build.gradle
```

---

## ⚙️ Technical Stack

- **Kotlin Version**: 1.5.21+
- **Android SDK**: API Level 17+
- **Gradle**: 7.0.2+
- **Libraries**:
  - Retrofit 2.9.0
  - OkHttp 4.9.3
  - Gson 2.8.8

---

## 🔧 Core Components

### 🏁 MainActivity.kt
Handles:
- UI rendering
- File picker intent
- API calls

---

### 🌐 ApiService.kt

```kotlin
interface ApiService {
    @Multipart
    @POST("api/transcribe")
    fun transcribeAudio(@Part file: MultipartBody.Part): Call<TranscriptionResponse>

    @POST("api/analyze_phishing")
    fun analyzePhishing(@Body request: PhishingAnalysisRequest): Call<PhishingAnalysisRequest>
}
```

---

## 📁 5.2 Core Components Explained

### 🧰 FileUtil.kt Analysis

The utility class handles secure file operations:

```kotlin
object FileUtil {
    // Creates temp file from Content URI
    fun getFileFromUri(context: Context, uri: Uri): File {
        val inputStream = context.contentResolver.openInputStream(uri)
        val fileName = getFileName(context, uri)
        val file = File(context.cacheDir, fileName)

        // Secure stream handling
        inputStream.use { input ->
            FileOutputStream(file).use { output ->
                input?.copyTo(output)
            }
        }
        return file
    }

    // Extracts original filename
    private fun getFileName(context: Context, uri: Uri): String {
        var result: String? = null
        if (uri.scheme == "content") {
            context.contentResolver.query(uri, null, null, null, null)?.use { cursor ->
                if (cursor.moveToFirst()) {
                    val nameIndex = cursor.getColumnIndex(OpenableColumns.DISPLAY_NAME)
                    if (nameIndex >= 0) {
                        result = cursor.getString(nameIndex)
                    }
                }
            }
        }
        if (result == null) {
            result = uri.path
            val cut = result?.lastIndexOf('/')
            if (cut != null && cut != -1) {
                result = result?.substring(cut + 1)
            }
        }
        return result ?: "audio_file_${System.currentTimeMillis()}.mp3"
    }
}
```

**Key Security Features:**
- Auto-closing streams (use blocks)
- Cache directory storage
- ContentResolver metadata extraction
- Fallback naming convention

---

## 🌍 5.3 API Integration Details

### 🔧 RetrofitClient.kt

The networking singleton implements:

```kotlin
object RetrofitClient {
    private const val BASE_URL = "https://node-bk-green.vercel.app/"

    private val loggingInterceptor = HttpLoggingInterceptor().apply {
        level = HttpLoggingInterceptor.Level.BODY
    }

    private val okHttpClient = OkHttpClient.Builder()
        .addInterceptor(loggingInterceptor)
        .connectTimeout(60, TimeUnit.SECONDS)
        .readTimeout(60, TimeUnit.SECONDS)
        .writeTimeout(60, TimeUnit.SECONDS)
        .build()

    private val retrofit = Retrofit.Builder()
        .baseUrl(BASE_URL)
        .client(okHttpClient)
        .addConverterFactory(GsonConverterFactory.create())
        .build()

    val apiService: ApiService = retrofit.create(ApiService::class.java)
}
```

**Performance Considerations:**
- 60-second timeouts for large audio
- Request/response logging
- GSON serialization optimization

---

## 🧩 5.4 Data Models

### 📄 TranscriptionResponse.kt

```kotlin
data class TranscriptionResponse(
    @SerializedName("text") val transcription: String,
    @SerializedName("warnings") val warnings: List<String>? = null
)
```

### 📄 PhishingAnalysisRequest.kt

```kotlin
data class PhishingAnalysisRequest(
    val text: String? = null,
    val success: Boolean? = null,
    val analysis: List<List<AnalysisResult>>? = null,
    val timestamp: String? = null
) {
    data class AnalysisResult(
        val label: String,
        val score: Double
    )
}
```

---

## 🔐 6. Security Considerations

- HTTPS with certificate pinning
- Cache directory isolation
- Input sanitization
- Minimal permissions (`READ_EXTERNAL_STORAGE` only)

---

## ⚡ 7. Performance Optimization

- File chunking for large audio
- Memory-efficient streaming
- Background processing
- Result caching

---

## 🧱 8. Build and Deployment

### 🔧 8.1 Gradle Configuration

```gradle
dependencies {
    implementation 'com.squareup.retrofit2:retrofit:2.9.0'
    implementation 'com.squareup.okhttp3:okhttp:4.9.3'
    implementation 'com.google.code.gson:gson:2.8.8'
}
```

### 🚀 8.2 CI/CD Pipeline

- GitHub Actions for builds
- Firebase App Distribution
- Play Store deployment

---

## 🧪 9. Testing Strategy

- Unit Tests (JUnit)
- Instrumentation Tests (Espresso)
- API Mocking (MockWebServer)
- Security Penetration Testing

---

## 🔮 10. Future Enhancements

- Real-time audio analysis
- Multi-language support
- Enhanced visualization
- User reporting system

---

## ✅ 11. Conclusion

SecureEcho represents a robust solution to modern audio phishing threats, combining advanced NLP with intuitive mobile interfaces.
