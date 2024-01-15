# Practica1_GPS
Objetivo. Gestionar los permisos de localización con una precisión definida y obtendremos la latitud y longitud donde se ubica el dispositivo.

Instalar la dependencia de localización de Google Play Services en el archivo build.gradle.kts ubicado en el apartado de Gradle Scripts, en el apartado de dependencies:
implementation ("com.google.android.gms:play-services-location:21.0.1")

Necesitamos pedir permiso para usar la localización, por lo que dentro del archivo AndroidManifest.xml, agregar las siguientes lineas dentro de la etiqueta :
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/>
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>

<application
        android:allowBackup="true"
        android:fullBackupContent="@xml/backup_rules"
        tools:targetApi="31"
        android:dataExtractionRules="@xml/data_extraction_rules"
        android:label="@string/app_name"
        android:icon="@mipmap/ic_launcher"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.GPS">

    <activity android:name=".MainActivity" tools:ignore="WrongManifestParent" android:exported="true">
        <intent-filter android:exported="true">
            <action android:name="android.intent.action.MAIN" />
            <category android:name="android.intent.category.LAUNCHER" />
        </intent-filter>
    </activity>

</application>
En el Layout Principal, añadir los siguientes Views que seran dos textviews que guardaran la latitud y longitud de la localizacion al presionar un boton:
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android" xmlns:app="http://schemas.android.com/apk/res-auto" xmlns:tools="http://schemas.android.com/tools" android:layout_width="match_parent" android:layout_height="match_parent" tools:context=".MainActivity"> <androidx.constraintlayout.widget.Guideline android:id="@+id/guideline" android:layout_width="wrap_content" android:layout_height="wrap_content" android:orientation="horizontal" app:layout_constraintGuide_percent="0.65" /> <androidx.constraintlayout.widget.Guideline android:id="@+id/guideline2" android:layout_width="wrap_content" android:layout_height="wrap_content" android:orientation="horizontal" app:layout_constraintGuide_percent="0.25" /> </androidx.constraintlayout.widget.ConstraintLayout>

En la clase Kotlin, seteamos las variables un id de permiso y el cliente localizador, asi como las variables de los Views:
package com.example.gps

import android.Manifest import android.os.Bundle import androidx.activity.ComponentActivity import android.widget.TextView import android.widget.Button import com.google.android.gms.location.FusedLocationProviderClient import androidx.core.app.ActivityCompat import android.content.pm.PackageManager import android.location.LocationManager import android.content.Context import android.annotation.SuppressLint import com.google.android.gms.location.LocationServices

@SuppressLint("RestrictedApi") class MainActivity: ComponentActivity(){

private lateinit var mFusedLocationClient: FusedLocationProviderClient
private lateinit var tvLatitude: TextView
private lateinit var tvLongitude: TextView
private lateinit var btnLocate: Button

companion object {
    const val PERMISSION_ID = 33
}

override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_main)

    // Inicializar vistas después de inflar la interfaz de usuario
    tvLatitude = findViewById(R.id.tvLatitude)
    tvLongitude = findViewById(R.id.tvLongitude)
    btnLocate = findViewById(R.id.btnLocate)

    // Inicializar FusedLocationProviderClient
    mFusedLocationClient = LocationServices.getFusedLocationProviderClient(this)

    btnLocate.setOnClickListener {
        getLocation()
    }
}



private fun checkGranted(permission: String): Boolean {
    return ActivityCompat.checkSelfPermission(this, permission) == PackageManager.PERMISSION_GRANTED
}

private fun checkPermissions() = checkGranted(Manifest.permission.ACCESS_COARSE_LOCATION) && checkGranted(Manifest.permission.ACCESS_FINE_LOCATION)

private fun requestPermissions() {
        ActivityCompat.requestPermissions(this, arrayOf(Manifest.permission.ACCESS_COARSE_LOCATION, Manifest.permission.ACCESS_FINE_LOCATION), PERMISSION_ID)
    }

private fun isLocationEnabled(): Boolean {
    val locationManager: LocationManager = getSystemService(Context.LOCATION_SERVICE) as LocationManager
    return locationManager.isProviderEnabled(LocationManager.GPS_PROVIDER) || locationManager.isProviderEnabled(LocationManager.NETWORK_PROVIDER)
}

@SuppressLint("MissingPermission")
private fun getLocation() {
    if (checkPermissions()) {
        if (isLocationEnabled()) {
            mFusedLocationClient.lastLocation.addOnSuccessListener(this) { location ->
                tvLatitude.text = location?.latitude.toString()
                tvLongitude.text = location?.longitude.toString()
            }
        }
    } else {
        requestPermissions()
    }
}
}

Creamos un metodo para consultar el status de algun permiso en la app.
private fun checkGranted(permission: String): Boolean{ return ActivityCompat.checkSelfPermission(this, permission) == PackageManager.PERMISSION_GRANTED } 6. Un codigo para checar si la app tiene permisos de localizacion. private fun checkPermissions() = checkGranted(Manifest.permission.ACCESS_COARSE_LOCATION) && checkGranted(Manifest.permission.ACCESS_FINE_LOCATION)

Una function para pedir el permiso de localizacion por si aun no se lo otorgamos en la app.
private fun requestPermissions() { ActivityCompat.requestPermissions( this, arrayOf(Manifest.permission.ACCESS_COARSE_LOCATION, Manifest.permission.ACCESS_FINE_LOCATION), PERMISSION_ID ) }

Consultamos si el GPS esta prendido.
private fun isLocationEnabled(): Boolean { var locationManager: LocationManager = getSystemService(Context.LOCATION_SERVICE) as LocationManager return locationManager.isProviderEnabled(LocationManager.GPS_PROVIDER) || locationManager.isProviderEnabled( LocationManager.NETWORK_PROVIDER )

}

Mostramos la localizacion en formato latitud y longitud en los TextViews correspondientes y asignamos el metodo getLocation al boton de localizacion.
@SuppressLint("MissingPermission") private fun getLocation() { if (checkPermissions()) { if (isLocationEnabled()) { mFusedLocationClient.lastLocation.addOnSuccessListener(this) { location -

tvLatitude.text = location?.latitude.toString() tvLongitude.text = location?.longitude.toString() } } } else{ requestPermissions() } }
