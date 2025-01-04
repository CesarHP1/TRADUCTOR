# TRADUCTOR
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">

    <uses-permission android:name="android.permission.INTERNET"/>

    <application
        android:allowBackup="true"
        android:dataExtractionRules="@xml/data_extraction_rules"
        android:fullBackupContent="@xml/backup_rules"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:usesCleartextTraffic="true"
        android:theme="@style/Theme.Traductor"
        tools:targetApi="31">
        <activity
            android:name=".MainActivity"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

</manifest>
package com.example.traductor;

import android.app.ProgressDialog;
import android.os.Bundle;
import android.util.Log;
import android.view.Menu;
import android.view.MenuItem;
import android.view.View;
import android.widget.EditText;
import android.widget.PopupMenu;
import android.widget.TextView;
import android.widget.Toast;

import androidx.annotation.NonNull;
import androidx.appcompat.app.AppCompatActivity;
import androidx.core.graphics.Insets;
import androidx.core.view.ViewCompat;
import androidx.core.view.WindowInsetsCompat;

import com.example.traductor.Modelo.Idioma;
import com.google.android.gms.tasks.OnFailureListener;
import com.google.android.gms.tasks.OnSuccessListener;
import com.google.android.material.button.MaterialButton;
import com.google.mlkit.common.model.DownloadConditions;
import com.google.mlkit.nl.translate.TranslateLanguage;
import com.google.mlkit.nl.translate.Translation;
import com.google.mlkit.nl.translate.Translator;
import com.google.mlkit.nl.translate.TranslatorOptions;

import java.util.ArrayList;
import java.util.List;
import java.util.Locale;

public class MainActivity extends AppCompatActivity {

    EditText Et_Idioma_Origen;
    TextView Tv_Idioma_Destino;
    MaterialButton Btn_Elegir_idioma, Btn_Idioma_Elegido, Btn_Traducir;
    private ProgressDialog progressDialog;

    private ArrayList<Idioma> IdiomasArrayList;

    private static final String REGISTROS = "Mis_registros";

    private String codigo_idioma_origen = "es";
    private String titulo_idioma_origen = "Español";
    private String codigo_idioma_destino = "en";
    private String titulo_idioma_destino = "Inglés";

    private TranslatorOptions translatorOptions;
    private Translator translator;
    private String Texto_idioma_origen = "";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        inicializarVistas();
        IdiomasDisponibles();

        ViewCompat.setOnApplyWindowInsetsListener(findViewById(R.id.main), (v, insets) -> {
            Insets systemBars = insets.getInsets(WindowInsetsCompat.Type.systemBars());
            v.setPadding(systemBars.left, systemBars.top, systemBars.right, systemBars.bottom);
            return insets;
        });

        // Configuración del OnClickListener para Btn_Elegir_idioma
        Btn_Elegir_idioma.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                ElegirIdiomaOrigen();
            }
        });

        // Configuración del OnClickListener para Btn_Idioma_Elegido (Inglés)
        Btn_Idioma_Elegido.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                ElegirIdiomaDestino();
            }
        });

        // Configuración del OnClickListener para Btn_Traducir
        Btn_Traducir.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                ValidarDatos();
            }
        });
    }

    private void inicializarVistas() {
        Et_Idioma_Origen = findViewById(R.id.Et_Idioma_Origen);
        Tv_Idioma_Destino = findViewById(R.id.Tv_Idioma_Destino);
        Btn_Elegir_idioma = findViewById(R.id.Btn_Elegir_idioma);
        Btn_Idioma_Elegido = findViewById(R.id.Btn_Idioma_Elegido);
        Btn_Traducir = findViewById(R.id.Btn_Traducir);
        progressDialog = new ProgressDialog(this);
        progressDialog.setTitle("Espere por favor");
        progressDialog.setCanceledOnTouchOutside(false);
    }

    private void IdiomasDisponibles() {
        IdiomasArrayList = new ArrayList<>();
        List<String> ListaCodigoIdioma = TranslateLanguage.getAllLanguages();

        for (String codigo_Lenguaje : ListaCodigoIdioma) {
            String titulo_Lenguaje = new Locale(codigo_Lenguaje).getDisplayLanguage();

            Idioma modeloIdioma = new Idioma(codigo_Lenguaje, titulo_Lenguaje);
            IdiomasArrayList.add(modeloIdioma);
        }
    }

    private void ElegirIdiomaOrigen() {
        PopupMenu popupMenu = new PopupMenu(this, Btn_Elegir_idioma);

        // Agregar los ítems al menú emergente
        for (int i = 0; i < IdiomasArrayList.size(); i++) {
            popupMenu.getMenu().add(Menu.NONE, i, i, IdiomasArrayList.get(i).getTitulo_idioma());
        }

        // Mostrar el menú emergente
        popupMenu.show();

        // Manejar la selección del ítem del menú
        popupMenu.setOnMenuItemClickListener(new PopupMenu.OnMenuItemClickListener() {
            @Override
            public boolean onMenuItemClick(MenuItem item) {
                int position = item.getItemId();

                codigo_idioma_origen = IdiomasArrayList.get(position).getCodigo_idioma();
                titulo_idioma_origen = IdiomasArrayList.get(position).getTitulo_idioma();

                Btn_Elegir_idioma.setText(titulo_idioma_origen);
                Et_Idioma_Origen.setHint("Ingrese texto en: " + titulo_idioma_origen);

                Log.d(REGISTROS, "onMenuItemClick: codigo_idioma_origen: " + codigo_idioma_origen);
                Log.d(REGISTROS, "onMenuItemClick: titulo_idioma_origen: " + titulo_idioma_origen);

                return true; // Devolver true para indicar que el evento ha sido consumido
            }
        });
    }

    private void ElegirIdiomaDestino() {
        PopupMenu popupMenu = new PopupMenu(this, Btn_Idioma_Elegido);

        // Agregar los ítems al menú emergente
        for (int i = 0; i < IdiomasArrayList.size(); i++) {
            popupMenu.getMenu().add(Menu.NONE, i, i, IdiomasArrayList.get(i).getTitulo_idioma());
        }

        // Mostrar el menú emergente
        popupMenu.show();

        // Manejar la selección del ítem del menú
        popupMenu.setOnMenuItemClickListener(new PopupMenu.OnMenuItemClickListener() {
            @Override
            public boolean onMenuItemClick(MenuItem item) {
                int position = item.getItemId();

                codigo_idioma_destino = IdiomasArrayList.get(position).getCodigo_idioma();
                titulo_idioma_destino = IdiomasArrayList.get(position).getTitulo_idioma();

                Btn_Idioma_Elegido.setText(titulo_idioma_destino);
                Tv_Idioma_Destino.setText("Texto traducido al: " + titulo_idioma_destino);

                Log.d(REGISTROS, "onMenuItemClick: codigo_idioma_destino: " + codigo_idioma_destino);
                Log.d(REGISTROS, "onMenuItemClick: titulo_idioma_destino: " + titulo_idioma_destino);

                return true; // Devolver true para indicar que el evento ha sido consumido
            }
        });
    }

    private void ValidarDatos() {
        Texto_idioma_origen = Et_Idioma_Origen.getText().toString().trim();
        Log.d(REGISTROS, "ValidarDatos: Texto_idioma_origen " + Texto_idioma_origen);

        if (Texto_idioma_origen.isEmpty()) {
            Toast.makeText(this, "Ingrese texto", Toast.LENGTH_SHORT).show();
        } else {
            TraducirTexto();
        }
    }

    private void TraducirTexto() {
        progressDialog.setMessage("Procesando");
        progressDialog.show();

        translatorOptions = new TranslatorOptions.Builder()
                .setSourceLanguage(codigo_idioma_origen)
                .setTargetLanguage(codigo_idioma_destino)
                .build();

        translator = Translation.getClient(translatorOptions);

        DownloadConditions downloadConditions = new DownloadConditions.Builder()
                .requireWifi()
                .build();

        translator.downloadModelIfNeeded(downloadConditions)
                .addOnSuccessListener(new OnSuccessListener<Void>() {
                    @Override
                    public void onSuccess(Void unused) {
                        Log.d(REGISTROS, "onSuccess: El paquete se ha descargado con éxito");
                        progressDialog.setMessage("Traduciendo texto");

                        translator.translate(Texto_idioma_origen)
                                .addOnSuccessListener(new OnSuccessListener<String>() {
                                    @Override
                                    public void onSuccess(String texto_traducido) {
                                        progressDialog.dismiss();
                                        Log.d(REGISTROS, "onSuccess: texto_traducido " + texto_traducido);
                                        Tv_Idioma_Destino.setText(texto_traducido);
                                    }
                                })
                                .addOnFailureListener(new OnFailureListener() {
                                    @Override
                                    public void onFailure(@NonNull Exception e) {
                                        progressDialog.dismiss();
                                        Log.d(REGISTROS, "onFailure: " + e);
                                        Toast.makeText(MainActivity.this, "" + e, Toast.LENGTH_SHORT).show();
                                    }
                                });
                    }
                })
                .addOnFailureListener(new OnFailureListener() {
                    @Override
                    public void onFailure(@NonNull Exception e) {
                        progressDialog.dismiss();
                        Log.d(REGISTROS, "onFailure: " + e);
                        Toast.makeText(MainActivity.this, "" + e, Toast.LENGTH_SHORT).show();
                    }
                });
    }

    private void mostrarMensaje(String mensaje) {
        Toast.makeText(MainActivity.this, mensaje, Toast.LENGTH_SHORT).show();
    }
}

package com.example.traductor;

import android.content.Context;

import androidx.test.platform.app.InstrumentationRegistry;
import androidx.test.ext.junit.runners.AndroidJUnit4;

import org.junit.Test;
import org.junit.runner.RunWith;

import static org.junit.Assert.*;

/**
 * Instrumented test, which will execute on an Android device.
 *
 * @see <a href="http://d.android.com/tools/testing">Testing documentation</a>
 */
@RunWith(AndroidJUnit4.class)
public class ExampleInstrumentedTest {
    @Test
    public void useAppContext() {
        // Context of the app under test.
        Context appContext = InstrumentationRegistry.getInstrumentation().getTargetContext();
        assertEquals("com.example.traductor", appContext.getPackageName());
    }
}
package com.example.traductor;

import org.junit.Test;

import static org.junit.Assert.*;

/**
 * Example local unit test, which will execute on the development machine (host).
 *
 * @see <a href="http://d.android.com/tools/testing">Testing documentation</a>
 */
public class ExampleUnitTest {
    @Test
    public void addition_isCorrect() {
        assertEquals(4, 2 + 2);
    }
}
