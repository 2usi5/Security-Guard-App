# Security-Guard-App
📁 1. MainActivity.kt (الكود الرئيسي)
package com.yusuf.securityscanner

import android.content.pm.PackageInfo
import android.content.pm.PackageManager
import android.net.wifi.WifiManager
import android.os.Bundle
import android.widget.*
import androidx.appcompat.app.AppCompatActivity

class MainActivity : AppCompatActivity() {

    lateinit var email: EditText
    lateinit var password: EditText
    lateinit var link: EditText
    lateinit var scanBtn: Button
    lateinit var result: TextView

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        email = findViewById(R.id.email)
        password = findViewById(R.id.password)
        link = findViewById(R.id.link)
        scanBtn = findViewById(R.id.scanBtn)
        result = findViewById(R.id.result)

        NotificationHelper.createChannel(this)

        scanBtn.setOnClickListener {
            runSecurityScan()
        }
    }

    private fun runSecurityScan() {

        val emailText = email.text.toString()
        val passText = password.text.toString()
        val linkText = link.text.toString()

        val log = StringBuilder()

        // Email Check
        if (SecurityUtils.isPhishingEmail(emailText)) {
            log.append("⚠ Phishing Email Detected\n")
            NotificationHelper.send(this,"Warning","Phishing Email")
        }

        // Password Check
        val passStrength = SecurityUtils.checkPassword(passText)
        log.append("Password: $passStrength\n")

        // Link Check
        if (SecurityUtils.isMaliciousLink(linkText)) {
            log.append("🚫 Unsafe Link Blocked\n")
            NotificationHelper.send(this,"Danger","Malicious Link")
        }

        // Apps Check
        val pm: PackageManager = packageManager
        val packages: List<PackageInfo> = pm.getInstalledPackages(0)

        for (app in packages) {
            if (app.packageName.contains("hack") || app.packageName.contains("spy")) {
                log.append("⚠ Suspicious App: ${app.packageName}\n")
            }
        }

        // WiFi Check
        val wifi = applicationContext.getSystemService(WIFI_SERVICE) as WifiManager
        if (wifi.isWifiEnabled) {
            log.append("WiFi: Enabled\n")
        } else {
            log.append("WiFi: Disabled\n")
        }

        result.text = log.toString()
    }
}

📁 2. SecurityUtils.kt
package com.yusuf.securityscanner

object SecurityUtils {

    fun isPhishingEmail(text: String): Boolean {
        val words = listOf("verify", "urgent", "login", "password")
        return words.any { text.contains(it, true) }
    }

    fun isMaliciousLink(url: String): Boolean {
        val keywords = listOf("hack", "phishing", "malware")
        return keywords.any { url.contains(it, true) }
    }

    fun checkPassword(password: String): String {
        return when {
            password.length < 6 -> "Weak"
            password.matches(".*[A-Z].*".toRegex()) &&
            password.matches(".*\\d.*".toRegex()) -> "Strong"
            else -> "Medium"
        }
    }
}

📁 3. NotificationHelper.kt
package com.yusuf.securityscanner

import android.app.*
import android.content.Context
import android.os.Build
import androidx.core.app.NotificationCompat

object NotificationHelper {

    private const val CHANNEL_ID = "SECURITY_CHANNEL"

    fun createChannel(context: Context) {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            val channel = NotificationChannel(
                CHANNEL_ID,
                "Security Alerts",
                NotificationManager.IMPORTANCE_HIGH
            )
            val manager = context.getSystemService(NotificationManager::class.java)
            manager.createNotificationChannel(channel)
        }
    }

    fun send(context: Context, title: String, msg: String) {
        val notification = NotificationCompat.Builder(context, CHANNEL_ID)
            .setContentTitle(title)
            .setContentText(msg)
            .setSmallIcon(android.R.drawable.ic_dialog_alert)
            .build()

        val manager = context.getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager
        manager.notify(System.currentTimeMillis().toInt(), notification)
    }
}

📁 4. activity_main.xml (الواجهة)
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:padding="16dp"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <EditText
        android:id="@+id/email"
        android:hint="Enter Email"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>

    <EditText
        android:id="@+id/password"
        android:hint="Enter Password"
        android:inputType="textPassword"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>

    <EditText
        android:id="@+id/link"
        android:hint="Enter Link"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>

    <Button
        android:id="@+id/scanBtn"
        android:text="Scan Security"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>

    <TextView
        android:id="@+id/result"
        android:text="Results"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>

</LinearLayout>

📁 5. AndroidManifest.xml (الصلاحيات)
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE"/>
<uses-permission android:name="android.permission.INTERNET"/>
<uses-permission android:name="android.permission.POST_NOTIFICATIONS"/>
