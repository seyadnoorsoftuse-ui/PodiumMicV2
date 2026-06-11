# 📋 PodiumMic v2.0 — GitHub Copy-Paste Guide

**முறை:** Repo-ல் ஒவ்வொரு file-க்கும்: `Add file → Create new file` → கீழே உள்ள **path-ஐ அப்படியே** file name box-ல் type செய்யுங்கள் (slash போட்டால் folders தானாக உருவாகும்) → content-ஐ paste → Commit changes.

**வரிசை முக்கியம் இல்லை, ஆனால் build.yml-ஐ கடைசியாக commit செய்தால் ஒரே build ஓடும்.**

---

## File 1 — File name box-ல் type செய்யவும்:

```
settings.gradle.kts
```

**Content (இதை paste செய்யுங்கள்):**

~~~~
pluginManagement {
    repositories {
        google()
        mavenCentral()
        gradlePluginPortal()
    }
}
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()
    }
}
rootProject.name = "PodiumMic"
include(":app")

~~~~

---

## File 2 — File name box-ல் type செய்யவும்:

```
build.gradle.kts
```

**Content (இதை paste செய்யுங்கள்):**

~~~~
plugins {
    id("com.android.application") version "8.5.0" apply false
    id("org.jetbrains.kotlin.android") version "1.9.24" apply false
}

~~~~

---

## File 3 — File name box-ல் type செய்யவும்:

```
gradle.properties
```

**Content (இதை paste செய்யுங்கள்):**

~~~~
org.gradle.jvmargs=-Xmx2048m
android.useAndroidX=true
android.nonTransitiveRClass=true

~~~~

---

## File 4 — File name box-ல் type செய்யவும்:

```
app/build.gradle.kts
```

**Content (இதை paste செய்யுங்கள்):**

~~~~
plugins {
    id("com.android.application")
    id("org.jetbrains.kotlin.android")
}

android {
    namespace = "com.seyad.podiummic"
    compileSdk = 34

    defaultConfig {
        applicationId = "com.seyad.podiummic"
        minSdk = 24
        targetSdk = 34
        versionCode = 2
        versionName = "2.0"
    }

    buildTypes {
        release {
            isMinifyEnabled = false
        }
    }
    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_17
        targetCompatibility = JavaVersion.VERSION_17
    }
    kotlinOptions {
        jvmTarget = "17"
    }
    buildFeatures {
        viewBinding = true
    }
}

dependencies {
    implementation("androidx.core:core-ktx:1.13.1")
    implementation("androidx.appcompat:appcompat:1.7.0")
    implementation("com.google.android.material:material:1.12.0")
    implementation("androidx.constraintlayout:constraintlayout:2.1.4")
}

~~~~

---

## File 5 — File name box-ல் type செய்யவும்:

```
app/src/main/AndroidManifest.xml
```

**Content (இதை paste செய்யுங்கள்):**

~~~~
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android">

    <uses-permission android:name="android.permission.RECORD_AUDIO" />
    <uses-permission android:name="android.permission.BLUETOOTH" android:maxSdkVersion="30" />
    <uses-permission android:name="android.permission.BLUETOOTH_CONNECT" />
    <uses-permission android:name="android.permission.MODIFY_AUDIO_SETTINGS" />
    <uses-permission android:name="android.permission.FOREGROUND_SERVICE" />
    <uses-permission android:name="android.permission.FOREGROUND_SERVICE_MICROPHONE" />
    <uses-permission android:name="android.permission.POST_NOTIFICATIONS" />

    <uses-feature android:name="android.hardware.microphone" android:required="true" />

    <application
        android:allowBackup="true"
        android:label="@string/app_name"
        android:icon="@mipmap/ic_launcher"
        android:theme="@style/Theme.PodiumMic"
        android:supportsRtl="true">

        <activity
            android:name=".MainActivity"
            android:exported="true"
            android:screenOrientation="fullSensor"
            android:configChanges="orientation|screenSize|keyboardHidden">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

        <service
            android:name=".MicStreamService"
            android:exported="false"
            android:foregroundServiceType="microphone" />
    </application>
</manifest>

~~~~

---

## File 6 — File name box-ல் type செய்யவும்:

```
app/src/main/java/com/seyad/podiummic/MainActivity.kt
```

**Content (இதை paste செய்யுங்கள்):**

~~~~
package com.seyad.podiummic

import android.Manifest
import android.content.Context
import android.content.Intent
import android.content.SharedPreferences
import android.content.pm.PackageManager
import android.graphics.Bitmap
import android.graphics.Canvas
import android.graphics.Color
import android.graphics.pdf.PdfRenderer
import android.hardware.display.DisplayManager
import android.media.AudioDeviceInfo
import android.media.AudioManager
import android.net.Uri
import android.os.Build
import android.os.Bundle
import android.os.ParcelFileDescriptor
import android.view.View
import android.view.WindowManager
import android.widget.Toast
import androidx.activity.result.contract.ActivityResultContracts
import androidx.appcompat.app.AppCompatActivity
import androidx.core.app.ActivityCompat
import androidx.core.content.ContextCompat
import com.seyad.podiummic.databinding.ActivityMainBinding

class MainActivity : AppCompatActivity() {

    private lateinit var b: ActivityMainBinding
    private lateinit var prefs: SharedPreferences
    private var fontSize = 24f
    private var editMode = false
    private var autoScrolling = false

    // ===== PDF state =====
    private var pdfRenderer: PdfRenderer? = null
    private var pdfFd: ParcelFileDescriptor? = null
    private var pdfPage = 0
    private var pdfMode = false

    // ===== Projector =====
    private var projector: ProjectorPresentation? = null

    private val scrollRunnable = object : Runnable {
        override fun run() {
            if (autoScrolling) {
                b.notesScroll.smoothScrollBy(0, 2)
                b.notesScroll.postDelayed(this, 50)
            }
        }
    }

    private val openDoc =
        registerForActivityResult(ActivityResultContracts.OpenDocument()) { uri ->
            uri?.let { handleFile(it) }
        }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        b = ActivityMainBinding.inflate(layoutInflater)
        setContentView(b.root)

        // ===== 1. KEEP SCREEN ON =====
        window.addFlags(WindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON)

        prefs = getSharedPreferences("podium", Context.MODE_PRIVATE)
        fontSize = prefs.getFloat("fontSize", 24f)
        b.notesView.textSize = fontSize
        b.notesEdit.setText(prefs.getString("notes", getString(R.string.sample_notes)))
        b.notesView.text = b.notesEdit.text

        b.btnEdit.setOnClickListener {
            if (pdfMode) exitPdfMode()
            editMode = !editMode
            if (editMode) {
                b.notesEdit.visibility = View.VISIBLE
                b.notesScroll.visibility = View.GONE
                b.btnEdit.text = getString(R.string.done)
            } else {
                prefs.edit().putString("notes", b.notesEdit.text.toString()).apply()
                b.notesView.text = b.notesEdit.text
                b.notesEdit.visibility = View.GONE
                b.notesScroll.visibility = View.VISIBLE
                b.btnEdit.text = getString(R.string.edit)
                updateProjector()
            }
        }

        b.btnFontUp.setOnClickListener { changeFont(+2f) }
        b.btnFontDown.setOnClickListener { changeFont(-2f) }

        b.btnAutoScroll.setOnClickListener {
            autoScrolling = !autoScrolling
            b.btnAutoScroll.text =
                if (autoScrolling) getString(R.string.stop_scroll) else getString(R.string.auto_scroll)
            if (autoScrolling) b.notesScroll.post(scrollRunnable)
        }

        // ===== FILE OPEN (txt / pdf / docx / pptx) =====
        b.btnOpen.setOnClickListener {
            openDoc.launch(arrayOf(
                "text/plain", "text/*",
                "application/pdf",
                "application/vnd.openxmlformats-officedocument.wordprocessingml.document",
                "application/vnd.openxmlformats-officedocument.presentationml.presentation",
                "application/msword",
                "application/vnd.ms-powerpoint"
            ))
        }

        // ===== PDF navigation =====
        b.btnPrevPage.setOnClickListener { showPdfPage(pdfPage - 1) }
        b.btnNextPage.setOnClickListener { showPdfPage(pdfPage + 1) }

        // ===== PROJECTOR =====
        b.btnProject.setOnClickListener { toggleProjector() }

        // ===== MIC -> BLUETOOTH =====
        b.btnMic.setOnClickListener {
            if (MicStreamService.isRunning) stopMic() else startMicWithPermission()
        }
        b.gainSlider.addOnChangeListener { _, value, _ ->
            MicStreamService.gain = value
            b.gainLabel.text = getString(R.string.gain_fmt, value)
        }
        b.gainSlider.value = 2.0f
    }

    // ---------- FILE HANDLING ----------

    private fun handleFile(uri: Uri) {
        val name = FileLoader.fileName(this, uri)
        val ext = name.substringAfterLast('.', "").lowercase()
        try {
            when (ext) {
                "pdf" -> openPdf(uri, name)
                "docx" -> showLoadedText(FileLoader.loadDocx(this, uri), name)
                "pptx" -> showLoadedText(FileLoader.loadPptx(this, uri), name)
                "txt", "md", "csv", "log", "json", "xml", "html" ->
                    showLoadedText(FileLoader.loadPlainText(this, uri), name)
                "doc", "ppt" ->
                    Toast.makeText(this, R.string.old_format, Toast.LENGTH_LONG).show()
                else ->
                    showLoadedText(FileLoader.loadPlainText(this, uri), name)
            }
        } catch (e: Exception) {
            Toast.makeText(this, getString(R.string.open_failed, e.message), Toast.LENGTH_LONG).show()
        }
    }

    private fun showLoadedText(text: String, name: String) {
        if (text.isBlank()) {
            Toast.makeText(this, R.string.empty_file, Toast.LENGTH_LONG).show()
            return
        }
        exitPdfMode()
        b.notesEdit.setText(text)
        b.notesView.text = text
        prefs.edit().putString("notes", text).apply()
        b.notesScroll.scrollTo(0, 0)
        Toast.makeText(this, getString(R.string.loaded_fmt, name), Toast.LENGTH_SHORT).show()
        updateProjector()
    }

    // ---------- PDF ----------

    private fun openPdf(uri: Uri, name: String) {
        closePdf()
        pdfFd = contentResolver.openFileDescriptor(uri, "r")
        val fd = pdfFd ?: return
        pdfRenderer = PdfRenderer(fd)
        pdfMode = true
        editMode = false
        b.notesEdit.visibility = View.GONE
        b.notesScroll.visibility = View.GONE
        b.pdfView.visibility = View.VISIBLE
        b.pdfNav.visibility = View.VISIBLE
        b.btnEdit.text = getString(R.string.edit)
        Toast.makeText(this, getString(R.string.loaded_fmt, name), Toast.LENGTH_SHORT).show()
        showPdfPage(0)
    }

    private fun showPdfPage(index: Int) {
        val renderer = pdfRenderer ?: return
        if (index < 0 || index >= renderer.pageCount) return
        pdfPage = index
        val bmp = renderPdfPage(index)
        b.pdfView.setImageBitmap(bmp)
        b.pageInfo.text = getString(R.string.page_fmt, index + 1, renderer.pageCount)
        projector?.showPage(bmp)
    }

    private fun renderPdfPage(index: Int): Bitmap {
        val renderer = pdfRenderer!!
        renderer.openPage(index).use { page ->
            val scale = 1600f / page.width
            val bmp = Bitmap.createBitmap(
                (page.width * scale).toInt(),
                (page.height * scale).toInt(),
                Bitmap.Config.ARGB_8888
            )
            Canvas(bmp).drawColor(Color.WHITE)
            page.render(bmp, null, null, PdfRenderer.Page.RENDER_MODE_FOR_DISPLAY)
            return bmp
        }
    }

    private fun exitPdfMode() {
        if (!pdfMode) return
        pdfMode = false
        closePdf()
        b.pdfView.visibility = View.GONE
        b.pdfNav.visibility = View.GONE
        b.notesScroll.visibility = View.VISIBLE
    }

    private fun closePdf() {
        try { pdfRenderer?.close() } catch (_: Exception) {}
        try { pdfFd?.close() } catch (_: Exception) {}
        pdfRenderer = null
        pdfFd = null
        pdfPage = 0
    }

    // ---------- PROJECTOR ----------

    private fun toggleProjector() {
        if (projector != null) {
            projector?.dismiss()
            projector = null
            b.btnProject.text = getString(R.string.project)
            return
        }
        val dm = getSystemService(Context.DISPLAY_SERVICE) as DisplayManager
        val displays = dm.getDisplays(DisplayManager.DISPLAY_CATEGORY_PRESENTATION)
        if (displays.isEmpty()) {
            Toast.makeText(this, R.string.no_display, Toast.LENGTH_LONG).show()
            return
        }
        projector = ProjectorPresentation(this, displays[0]).also {
            it.setOnDismissListener {
                projector = null
                b.btnProject.text = getString(R.string.project)
            }
            it.show()
        }
        b.btnProject.text = getString(R.string.project_stop)
        updateProjector()
    }

    private fun updateProjector() {
        val p = projector ?: return
        if (pdfMode && pdfRenderer != null) p.showPage(renderPdfPage(pdfPage))
        else p.showText(b.notesView.text, fontSize)
    }

    // ---------- FONT ----------

    private fun changeFont(delta: Float) {
        fontSize = (fontSize + delta).coerceIn(14f, 72f)
        b.notesView.textSize = fontSize
        b.notesEdit.textSize = fontSize
        prefs.edit().putFloat("fontSize", fontSize).apply()
        updateProjector()
    }

    // ---------- MIC ----------

    private fun startMicWithPermission() {
        val needed = mutableListOf(Manifest.permission.RECORD_AUDIO)
        if (Build.VERSION.SDK_INT >= 31) needed += Manifest.permission.BLUETOOTH_CONNECT
        if (Build.VERSION.SDK_INT >= 33) needed += Manifest.permission.POST_NOTIFICATIONS
        val notGranted = needed.filter {
            ContextCompat.checkSelfPermission(this, it) != PackageManager.PERMISSION_GRANTED
        }
        if (notGranted.isNotEmpty()) {
            ActivityCompat.requestPermissions(this, notGranted.toTypedArray(), 100)
            return
        }
        startMic()
    }

    override fun onRequestPermissionsResult(
        requestCode: Int, permissions: Array<out String>, grantResults: IntArray
    ) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults)
        if (requestCode == 100 && grantResults.isNotEmpty() &&
            grantResults[0] == PackageManager.PERMISSION_GRANTED
        ) startMic()
        else Toast.makeText(this, R.string.perm_needed, Toast.LENGTH_LONG).show()
    }

    private fun startMic() {
        if (!isBluetoothOutputConnected()) {
            Toast.makeText(this, R.string.connect_bt_first, Toast.LENGTH_LONG).show()
        }
        val i = Intent(this, MicStreamService::class.java)
        if (Build.VERSION.SDK_INT >= 26) startForegroundService(i) else startService(i)
        b.btnMic.text = getString(R.string.mic_stop)
        b.micStatus.text = getString(R.string.mic_live)
    }

    private fun stopMic() {
        stopService(Intent(this, MicStreamService::class.java))
        b.btnMic.text = getString(R.string.mic_start)
        b.micStatus.text = getString(R.string.mic_off)
    }

    private fun isBluetoothOutputConnected(): Boolean {
        val am = getSystemService(Context.AUDIO_SERVICE) as AudioManager
        return am.getDevices(AudioManager.GET_DEVICES_OUTPUTS).any {
            it.type == AudioDeviceInfo.TYPE_BLUETOOTH_A2DP ||
            it.type == AudioDeviceInfo.TYPE_BLUETOOTH_SCO
        }
    }

    override fun onResume() {
        super.onResume()
        if (MicStreamService.isRunning) {
            b.btnMic.text = getString(R.string.mic_stop)
            b.micStatus.text = getString(R.string.mic_live)
        }
    }

    override fun onDestroy() {
        projector?.dismiss()
        projector = null
        closePdf()
        super.onDestroy()
    }
}

~~~~

---

## File 7 — File name box-ல் type செய்யவும்:

```
app/src/main/java/com/seyad/podiummic/MicStreamService.kt
```

**Content (இதை paste செய்யுங்கள்):**

~~~~
package com.seyad.podiummic

import android.app.Notification
import android.app.NotificationChannel
import android.app.NotificationManager
import android.app.PendingIntent
import android.app.Service
import android.content.Context
import android.content.Intent
import android.media.AudioAttributes
import android.media.AudioFormat
import android.media.AudioManager
import android.media.AudioRecord
import android.media.AudioTrack
import android.media.MediaRecorder
import android.media.audiofx.AcousticEchoCanceler
import android.media.audiofx.NoiseSuppressor
import android.os.Build
import android.os.IBinder
import kotlin.concurrent.thread
import kotlin.math.max

/**
 * Phone mic-ஐ live-ஆக படித்து (AudioRecord), gain ஏற்றி,
 * AudioTrack வழியாக output-க்கு அனுப்புகிறது.
 * Bluetooth speaker/amplifier connect ஆகி இருந்தால்,
 * Android அதை default output-ஆக route செய்யும் (A2DP).
 */
class MicStreamService : Service() {

    companion object {
        @Volatile var isRunning = false
        @Volatile var gain = 2.0f          // 1.0 = normal, 4.0 = max boost
        const val SAMPLE_RATE = 44100
        const val CHANNEL_ID = "mic_stream"
    }

    private var worker: Thread? = null

    override fun onBind(intent: Intent?): IBinder? = null

    override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
        if (isRunning) return START_STICKY
        isRunning = true
        startForeground(1, buildNotification())
        startAudioLoop()
        return START_STICKY
    }

    private fun startAudioLoop() {
        worker = thread(name = "MicLoop") {
            val minRec = AudioRecord.getMinBufferSize(
                SAMPLE_RATE, AudioFormat.CHANNEL_IN_MONO, AudioFormat.ENCODING_PCM_16BIT
            )
            val minPlay = AudioTrack.getMinBufferSize(
                SAMPLE_RATE, AudioFormat.CHANNEL_OUT_MONO, AudioFormat.ENCODING_PCM_16BIT
            )
            val bufSize = max(minRec, minPlay)

            val recorder: AudioRecord
            try {
                recorder = AudioRecord(
                    MediaRecorder.AudioSource.MIC,
                    SAMPLE_RATE,
                    AudioFormat.CHANNEL_IN_MONO,
                    AudioFormat.ENCODING_PCM_16BIT,
                    bufSize * 2
                )
            } catch (e: SecurityException) {
                stopSelf(); return@thread
            }

            // Echo / noise reduction (device support இருந்தால்)
            if (AcousticEchoCanceler.isAvailable())
                AcousticEchoCanceler.create(recorder.audioSessionId)?.enabled = true
            if (NoiseSuppressor.isAvailable())
                NoiseSuppressor.create(recorder.audioSessionId)?.enabled = true

            val player = AudioTrack.Builder()
                .setAudioAttributes(
                    AudioAttributes.Builder()
                        .setUsage(AudioAttributes.USAGE_MEDIA)
                        .setContentType(AudioAttributes.CONTENT_TYPE_SPEECH)
                        .build()
                )
                .setAudioFormat(
                    AudioFormat.Builder()
                        .setEncoding(AudioFormat.ENCODING_PCM_16BIT)
                        .setSampleRate(SAMPLE_RATE)
                        .setChannelMask(AudioFormat.CHANNEL_OUT_MONO)
                        .build()
                )
                .setBufferSizeInBytes(bufSize * 2)
                .setTransferMode(AudioTrack.MODE_STREAM)
                .build()

            val am = getSystemService(Context.AUDIO_SERVICE) as AudioManager
            am.mode = AudioManager.MODE_NORMAL

            val buffer = ShortArray(bufSize / 2)
            recorder.startRecording()
            player.play()

            try {
                while (isRunning && !Thread.currentThread().isInterrupted) {
                    val read = recorder.read(buffer, 0, buffer.size)
                    if (read > 0) {
                        val g = gain
                        if (g != 1.0f) {
                            for (i in 0 until read) {
                                val v = (buffer[i] * g).toInt()
                                buffer[i] = v.coerceIn(
                                    Short.MIN_VALUE.toInt(),
                                    Short.MAX_VALUE.toInt()
                                ).toShort()
                            }
                        }
                        player.write(buffer, 0, read)
                    }
                }
            } finally {
                try { recorder.stop() } catch (_: Exception) {}
                recorder.release()
                try { player.stop() } catch (_: Exception) {}
                player.release()
            }
        }
    }

    private fun buildNotification(): Notification {
        val nm = getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager
        if (Build.VERSION.SDK_INT >= 26) {
            nm.createNotificationChannel(
                NotificationChannel(
                    CHANNEL_ID, getString(R.string.mic_channel),
                    NotificationManager.IMPORTANCE_LOW
                )
            )
        }
        val pi = PendingIntent.getActivity(
            this, 0, Intent(this, MainActivity::class.java),
            PendingIntent.FLAG_IMMUTABLE
        )
        val builder = if (Build.VERSION.SDK_INT >= 26)
            Notification.Builder(this, CHANNEL_ID) else Notification.Builder(this)
        return builder
            .setContentTitle(getString(R.string.app_name))
            .setContentText(getString(R.string.mic_live))
            .setSmallIcon(android.R.drawable.ic_btn_speak_now)
            .setContentIntent(pi)
            .setOngoing(true)
            .build()
    }

    override fun onDestroy() {
        isRunning = false
        worker?.interrupt()
        worker = null
        super.onDestroy()
    }
}

~~~~

---

## File 8 — File name box-ல் type செய்யவும்:

```
app/src/main/java/com/seyad/podiummic/FileLoader.kt
```

**Content (இதை paste செய்யுங்கள்):**

~~~~
package com.seyad.podiummic

import android.content.Context
import android.net.Uri
import android.provider.OpenableColumns
import java.io.BufferedReader
import java.io.InputStreamReader
import java.util.zip.ZipInputStream

/**
 * TXT / DOCX / PPTX files-ஐ plain text-ஆக மாற்றும் helper.
 * DOCX, PPTX = உள்ளே XML இருக்கும் zip files —
 * library இல்லாமல் நேரடியாக text எடுக்கிறோம்.
 */
object FileLoader {

    fun fileName(ctx: Context, uri: Uri): String {
        var name = "file"
        ctx.contentResolver.query(uri, null, null, null, null)?.use { c ->
            val idx = c.getColumnIndex(OpenableColumns.DISPLAY_NAME)
            if (idx >= 0 && c.moveToFirst()) name = c.getString(idx) ?: "file"
        }
        return name
    }

    /** Plain text / markdown / csv */
    fun loadPlainText(ctx: Context, uri: Uri): String {
        ctx.contentResolver.openInputStream(uri)?.use { ins ->
            return BufferedReader(InputStreamReader(ins, Charsets.UTF_8)).readText()
        }
        return ""
    }

    /** DOCX: word/document.xml -> paragraphs */
    fun loadDocx(ctx: Context, uri: Uri): String {
        val xml = readZipEntry(ctx, uri, "word/document.xml") ?: return ""
        var t = xml
            .replace(Regex("<w:br[^>]*/>"), "\n")
            .replace(Regex("<w:tab[^>]*/>"), "\t")
            .replace("</w:p>", "\n")
        t = t.replace(Regex("<[^>]+>"), "")
        return decodeEntities(t).lines()
            .joinToString("\n") { it.trimEnd() }
            .replace(Regex("\n{3,}"), "\n\n")
            .trim()
    }

    /** PPTX: ppt/slides/slideN.xml -> "— Slide N —" + text */
    fun loadPptx(ctx: Context, uri: Uri): String {
        val slides = sortedMapOf<Int, String>()
        ctx.contentResolver.openInputStream(uri)?.use { ins ->
            ZipInputStream(ins).use { zip ->
                var e = zip.nextEntry
                val re = Regex("ppt/slides/slide(\\d+)\\.xml")
                while (e != null) {
                    val m = re.matchEntire(e.name)
                    if (m != null) {
                        val num = m.groupValues[1].toInt()
                        slides[num] = zip.readBytes().toString(Charsets.UTF_8)
                    }
                    e = zip.nextEntry
                }
            }
        }
        if (slides.isEmpty()) return ""
        val sb = StringBuilder()
        for ((num, xml) in slides) {
            sb.append("━━━ Slide $num ━━━\n")
            // ஒவ்வொரு paragraph (<a:p>)-ஐயும் ஒரு line-ஆக
            val paras = xml.split("</a:p>")
            for (p in paras) {
                val texts = Regex("<a:t>(.*?)</a:t>", RegexOption.DOT_MATCHES_ALL)
                    .findAll(p).map { it.groupValues[1] }.joinToString("")
                if (texts.isNotBlank()) sb.append(decodeEntities(texts).trim()).append("\n")
            }
            sb.append("\n")
        }
        return sb.toString().trim()
    }

    private fun readZipEntry(ctx: Context, uri: Uri, entryName: String): String? {
        ctx.contentResolver.openInputStream(uri)?.use { ins ->
            ZipInputStream(ins).use { zip ->
                var e = zip.nextEntry
                while (e != null) {
                    if (e.name == entryName) return zip.readBytes().toString(Charsets.UTF_8)
                    e = zip.nextEntry
                }
            }
        }
        return null
    }

    private fun decodeEntities(s: String): String = s
        .replace("&amp;", "&").replace("&lt;", "<").replace("&gt;", ">")
        .replace("&quot;", "\"").replace("&apos;", "'").replace("&#10;", "\n")
}

~~~~

---

## File 9 — File name box-ல் type செய்யவும்:

```
app/src/main/java/com/seyad/podiummic/ProjectorPresentation.kt
```

**Content (இதை paste செய்யுங்கள்):**

~~~~
package com.seyad.podiummic

import android.app.Presentation
import android.content.Context
import android.graphics.Bitmap
import android.graphics.Color
import android.os.Bundle
import android.view.Display
import android.view.ViewGroup.LayoutParams.MATCH_PARENT
import android.widget.FrameLayout
import android.widget.ImageView
import android.widget.ScrollView
import android.widget.TextView

/**
 * Projector / HDMI / Wireless Display-ல் notes அல்லது PDF page-ஐ
 * தனித் திரையாக காட்டும் (phone-ல் controls, projector-ல் content).
 */
class ProjectorPresentation(outerContext: Context, display: Display) :
    Presentation(outerContext, display) {

    private var textView: TextView? = null
    private var imageView: ImageView? = null
    private var scroll: ScrollView? = null

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        val root = FrameLayout(context).apply { setBackgroundColor(Color.BLACK) }

        textView = TextView(context).apply {
            setTextColor(Color.WHITE)
            textSize = 40f
            setPadding(64, 48, 64, 48)
            setLineSpacing(0f, 1.4f)
        }
        scroll = ScrollView(context).apply { addView(textView) }

        imageView = ImageView(context).apply {
            scaleType = ImageView.ScaleType.FIT_CENTER
            visibility = android.view.View.GONE
        }

        root.addView(scroll, FrameLayout.LayoutParams(MATCH_PARENT, MATCH_PARENT))
        root.addView(imageView, FrameLayout.LayoutParams(MATCH_PARENT, MATCH_PARENT))
        setContentView(root)
    }

    fun showText(text: CharSequence, phoneFontSp: Float) {
        val tv = textView ?: return
        scroll?.visibility = android.view.View.VISIBLE
        imageView?.visibility = android.view.View.GONE
        tv.textSize = phoneFontSp * 1.6f   // projector-ல் பெரிதாக
        tv.text = text
    }

    fun showPage(bitmap: Bitmap) {
        val iv = imageView ?: return
        scroll?.visibility = android.view.View.GONE
        iv.visibility = android.view.View.VISIBLE
        iv.setImageBitmap(bitmap)
    }
}

~~~~

---

## File 10 — File name box-ல் type செய்யவும்:

```
app/src/main/res/layout/activity_main.xml
```

**Content (இதை paste செய்யுங்கள்):**

~~~~
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:background="#000000">

    <!-- Top control bar row 1 -->
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        android:paddingStart="8dp"
        android:paddingEnd="8dp"
        android:paddingTop="4dp"
        android:gravity="center_vertical"
        android:background="#111111">

        <Button
            android:id="@+id/btnEdit"
            style="@style/Widget.Material3.Button.TonalButton"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@string/edit" />

        <Button
            android:id="@+id/btnFontDown"
            style="@style/Widget.Material3.Button.TonalButton"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginStart="6dp"
            android:text="A-" />

        <Button
            android:id="@+id/btnFontUp"
            style="@style/Widget.Material3.Button.TonalButton"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginStart="4dp"
            android:text="A+" />

        <Button
            android:id="@+id/btnAutoScroll"
            style="@style/Widget.Material3.Button.TonalButton"
            android:layout_width="0dp"
            android:layout_weight="1"
            android:layout_height="wrap_content"
            android:layout_marginStart="6dp"
            android:text="@string/auto_scroll" />
    </LinearLayout>

    <!-- Top control bar row 2: Open file + Projector -->
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        android:paddingStart="8dp"
        android:paddingEnd="8dp"
        android:paddingBottom="4dp"
        android:gravity="center_vertical"
        android:background="#111111">

        <Button
            android:id="@+id/btnOpen"
            style="@style/Widget.Material3.Button.TonalButton"
            android:layout_width="0dp"
            android:layout_weight="1"
            android:layout_height="wrap_content"
            android:text="@string/open_file" />

        <Button
            android:id="@+id/btnProject"
            style="@style/Widget.Material3.Button.TonalButton"
            android:layout_width="0dp"
            android:layout_weight="1"
            android:layout_height="wrap_content"
            android:layout_marginStart="8dp"
            android:text="@string/project" />
    </LinearLayout>

    <!-- Notes view (read mode) -->
    <ScrollView
        android:id="@+id/notesScroll"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1"
        android:fillViewport="true">

        <TextView
            android:id="@+id/notesView"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:padding="20dp"
            android:textColor="#FFFFFF"
            android:textSize="24sp"
            android:lineSpacingMultiplier="1.4" />
    </ScrollView>

    <!-- Notes edit mode -->
    <EditText
        android:id="@+id/notesEdit"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1"
        android:visibility="gone"
        android:gravity="top|start"
        android:padding="20dp"
        android:textColor="#FFFFFF"
        android:textColorHint="#777777"
        android:hint="@string/notes_hint"
        android:background="#000000"
        android:inputType="textMultiLine"
        android:textSize="24sp" />

    <!-- PDF view -->
    <ImageView
        android:id="@+id/pdfView"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1"
        android:visibility="gone"
        android:scaleType="fitCenter"
        android:contentDescription="@string/pdf_page" />

    <!-- PDF navigation -->
    <LinearLayout
        android:id="@+id/pdfNav"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        android:visibility="gone"
        android:gravity="center_vertical"
        android:padding="6dp"
        android:background="#111111">

        <Button
            android:id="@+id/btnPrevPage"
            style="@style/Widget.Material3.Button.TonalButton"
            android:layout_width="0dp"
            android:layout_weight="1"
            android:layout_height="wrap_content"
            android:text="@string/prev_page" />

        <TextView
            android:id="@+id/pageInfo"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginStart="12dp"
            android:layout_marginEnd="12dp"
            android:textColor="#FFFFFF"
            android:text="1 / 1" />

        <Button
            android:id="@+id/btnNextPage"
            style="@style/Widget.Material3.Button.TonalButton"
            android:layout_width="0dp"
            android:layout_weight="1"
            android:layout_height="wrap_content"
            android:text="@string/next_page" />
    </LinearLayout>

    <!-- Bottom mic panel -->
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        android:padding="12dp"
        android:background="#111111">

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="horizontal"
            android:gravity="center_vertical">

            <Button
                android:id="@+id/btnMic"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="@string/mic_start" />

            <TextView
                android:id="@+id/micStatus"
                android:layout_width="0dp"
                android:layout_weight="1"
                android:layout_height="wrap_content"
                android:layout_marginStart="12dp"
                android:text="@string/mic_off"
                android:textColor="#AAAAAA" />
        </LinearLayout>

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="horizontal"
            android:gravity="center_vertical">

            <TextView
                android:id="@+id/gainLabel"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="@string/gain_default"
                android:textColor="#AAAAAA" />

            <com.google.android.material.slider.Slider
                android:id="@+id/gainSlider"
                android:layout_width="0dp"
                android:layout_weight="1"
                android:layout_height="wrap_content"
                android:layout_marginStart="8dp"
                android:valueFrom="1.0"
                android:valueTo="4.0"
                android:stepSize="0.1"
                app:labelBehavior="floating" />
        </LinearLayout>
    </LinearLayout>
</LinearLayout>

~~~~

---

## File 11 — File name box-ல் type செய்யவும்:

```
app/src/main/res/values/strings.xml
```

**Content (இதை paste செய்யுங்கள்):**

~~~~
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string name="app_name">Podium Mic</string>
    <string name="edit">Edit</string>
    <string name="done">Done ✓</string>
    <string name="auto_scroll">▶ Scroll</string>
    <string name="stop_scroll">⏸ Stop</string>
    <string name="notes_hint">உங்கள் பேச்சு குறிப்புகளை இங்கே எழுதுங்கள்…</string>
    <string name="sample_notes">வணக்கம்!\n\nஇங்கே உங்கள் மேடைப் பேச்சு குறிப்புகளை சேமிக்கலாம்.\n\n• Edit பட்டனை அழுத்தி notes-ஐ மாற்றுங்கள்\n• 📂 Open = TXT / PDF / DOCX / PPTX file திறக்க\n• 📽 Project = HDMI/Cast projector-ல் காட்ட\n• A+ / A- எழுத்து அளவு\n• ▶ Scroll = teleprompter\n• கீழே Mic ON செய்தால் Bluetooth speaker-ல் உங்கள் குரல் கேட்கும்\n\nதிரை தானாக அணையாது (Keep Screen On).</string>
    <string name="mic_start">🎤 Mic ON</string>
    <string name="mic_stop">⏹ Mic OFF</string>
    <string name="mic_off">Mic: Off</string>
    <string name="mic_live">Mic LIVE → Bluetooth output</string>
    <string name="mic_channel">Mic Streaming</string>
    <string name="gain_default">Gain: 2.0x</string>
    <string name="gain_fmt">Gain: %.1fx</string>
    <string name="perm_needed">Microphone permission தேவை</string>
    <string name="connect_bt_first">Bluetooth speaker/amplifier connect ஆகவில்லை — phone speaker-ல் ஒலிக்கும் (echo வரலாம்!)</string>
    <string name="open_file">📂 Open File</string>
    <string name="project">📽 Project</string>
    <string name="project_stop">📽 Stop</string>
    <string name="prev_page">◀ Prev</string>
    <string name="next_page">Next ▶</string>
    <string name="page_fmt">%1$d / %2$d</string>
    <string name="pdf_page">PDF page</string>
    <string name="old_format">பழைய .doc/.ppt format support இல்லை — file-ஐ .docx/.pptx ஆக "Save As" செய்யுங்கள்</string>
    <string name="empty_file">File-ல் text எதுவும் கிடைக்கவில்லை</string>
    <string name="open_failed">File திறக்க முடியவில்லை: %1$s</string>
    <string name="loaded_fmt">✓ %1$s loaded</string>
    <string name="no_display">Projector கண்டுபிடிக்கப்படவில்லை — HDMI/USB-C cable அல்லது Settings → Cast/Smart View-ல் முதலில் connect செய்யுங்கள்</string>
</resources>

~~~~

---

## File 12 — File name box-ல் type செய்யவும்:

```
app/src/main/res/values/themes.xml
```

**Content (இதை paste செய்யுங்கள்):**

~~~~
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <style name="Theme.PodiumMic" parent="Theme.Material3.Dark.NoActionBar">
        <item name="android:statusBarColor">#000000</item>
        <item name="android:navigationBarColor">#000000</item>
    </style>
</resources>

~~~~

---

## File 13 — File name box-ல் type செய்யவும்:

```
app/src/main/res/values/colors.xml
```

**Content (இதை paste செய்யுங்கள்):**

~~~~
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <color name="icon_bg">#1A237E</color>
</resources>

~~~~

---

## File 14 — File name box-ல் type செய்யவும்:

```
app/src/main/res/drawable/ic_mic_fg.xml
```

**Content (இதை paste செய்யுங்கள்):**

~~~~
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="108dp" android:height="108dp"
    android:viewportWidth="108" android:viewportHeight="108">
    <group android:scaleX="0.5" android:scaleY="0.5"
        android:translateX="27" android:translateY="27">
        <path android:fillColor="#FFFFFF"
            android:pathData="M54,66c9.9,0 18,-8.1 18,-18V24c0,-9.9 -8.1,-18 -18,-18s-18,8.1 -18,18v24C36,57.9 44.1,66 54,66zM84,48c0,16.5 -13.5,30 -30,30S24,64.5 24,48h-9c0,19.8 14.7,36.2 33.8,38.7V102h10.5V86.7C78.3,84.2 93,67.8 93,48H84z"/>
    </group>
</vector>

~~~~

---

## File 15 — File name box-ல் type செய்யவும்:

```
app/src/main/res/mipmap-anydpi-v26/ic_launcher.xml
```

**Content (இதை paste செய்யுங்கள்):**

~~~~
<?xml version="1.0" encoding="utf-8"?>
<adaptive-icon xmlns:android="http://schemas.android.com/apk/res/android">
    <background android:drawable="@color/icon_bg"/>
    <foreground android:drawable="@drawable/ic_mic_fg"/>
</adaptive-icon>

~~~~

---

## File 16 — File name box-ல் type செய்யவும்:

```
.github/workflows/build.yml
```

**Content (இதை paste செய்யுங்கள்):**

~~~~
name: Build APK

on:
  push:
    branches: [ main, master ]
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up JDK 17
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: '17'

      - name: Setup Gradle
        uses: gradle/actions/setup-gradle@v3
        with:
          gradle-version: '8.7'

      - name: Build Debug APK
        run: gradle assembleDebug

      - name: Upload APK
        uses: actions/upload-artifact@v4
        with:
          name: PodiumMic-APK
          path: app/build/outputs/apk/debug/app-debug.apk

~~~~

---
