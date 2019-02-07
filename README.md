# Belatrix Android Prueba Tecnica 
1. Propón los pasos, clases, layouts y recursos que utilizarías para hacer un Stepper reutilizable y que cumpla con los parámetros definidos en la guía de material design:
https://material.io/guidelines/components/steppers.html
Solución:
Pasos:
    1.	Crear un contenedor raíz LinearLayout el cual contendrá 3 secciones:
        * Steps, dentro de una vista personalizada que llamaremos Contenedor de Tabs en el cual cada tab actuara como un "Step" del Stepper.
        * Contenido, Framelayout en el cual residirá un ViewPager el cual contendrá la vista a mostrar en cada paso, esta vista puede ser datos a mostrar, acciones a realizar, etc. Hay que recordar que el ViewPager responderá a la selección del "Step" en el contenedor de tabs o a los botones Anterior, Continuar o Finalizar.
        * Navegación, RelativeLayout que contiene los botones "Anterior", "Siguiente" y "Finalizar".
    2.	Crear la vista personalizada del Contenedor de Tabs, el cual tendrá dentro un HorizontalScrollView sea el caso la cantidad de steps supere la capacidad de la pantalla y se necesite deslizar el control para navegar entre cada Tab, recordar que cada Tab representa a un step y es necesario tener una vista completa de estos steps y su descripción. Para crear esta vista será necesario definir lo siguiente:
    2.1 El layout del tab o step, el cual contiene el "stepper circle" preparado para mostrarse activo o inactivo de tamaño 24dpx24dp, sea cambiando de color de background o de imagen de fondo, con un número del step actual encima del tipo TextView de Font Roboto Regular con tamaño 12sp. A lo anterior con una separación de 8dp se suma el título y subtitulo los cuales serán TextView dentro de un linear layout de color black al 87% con font del tipo Roboto Medium de tamaño 14sp. La altura de este layout debe ser de 72dp con un padding de 24dp para top, bottom y left.
    2.2 Crear el view model o clase que contendrá la información de cada step, el cual nos indicará: Titulo, Subtitulo y si ¿El paso es opcional?
    2.3 La clase que contendrá la vista personalizada, la cual se encargará de crear los steps en base a una lista de objetos o viewmodel de cada step. Se encargará de actualizarlos de activo a inactivo, también de manejar los Listener para actualizar el ViewPager de contenido también moverá el HorizontalScrollView para mostrar en primer lugar el Step actual.

2. Crea un shake action en android y pon el código.
```java
package com.fleetmode.fat.activity;

import android.app.Activity;
import android.hardware.Sensor;
import android.hardware.SensorEvent;
import android.hardware.SensorEventListener;
import android.hardware.SensorManager;
import android.os.Bundle;
import android.view.View;
import android.view.ViewGroup;
import android.widget.TextView;

public class ShakeAction extends Activity implements SensorEventListener {
    private SensorManager sensorManager;
    private Sensor accelerometer;
    private long prevUpdate = -1;
    private float x, y, z;
    private float prev_x, prev_y, prev_z;
    private static final int SHAKE_UMBRAL = 800;
    TextView tv;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        sensorManager = (SensorManager) getSystemService(SENSOR_SERVICE);
        accelerometer = sensorManager.getDefaultSensor(Sensor.TYPE_ACCELEROMETER);
        if (accelerometer != null) {
            sensorManager.registerListener(this, accelerometer,
                    SensorManager.SENSOR_DELAY_FASTEST);
        }

        tv = new TextView(this);
        tv.setGravity(View.TEXT_ALIGNMENT_CENTER);
        tv.setText("Shake me please");
        setContentView(tv, new ViewGroup.LayoutParams(ViewGroup.LayoutParams.MATCH_PARENT, ViewGroup.LayoutParams.MATCH_PARENT));
    }

    @Override
    public void onSensorChanged(SensorEvent sensorEvent) {
        if (sensorEvent.sensor.getType() == Sensor.TYPE_ACCELEROMETER) {
            long curTime = System.currentTimeMillis();

            if ((curTime - prevUpdate) > 100) {
                long diffTime = (curTime - prevUpdate);
                prevUpdate = curTime;

                x = sensorEvent.values[0];
                y = sensorEvent.values[1];
                z = sensorEvent.values[2];

                float speed = Math.abs(x+y+z - prev_x - prev_y - prev_z)
                        / diffTime * 10000;
                if (speed > SHAKE_UMBRAL) {
                    tv.setText("Shaked!");
                }
                prev_x = x;
                prev_y = y;
                prev_z = z;
            }
        }
    }

    @Override
    public void onAccuracyChanged(Sensor sensor, int i) {

    }
}

```

 
3. Explica cómo organizas en base a tu experiencia un proyecto en Android utilizando MVP e implementando Clean Architecture, menciona los paquetes que utilizarías y la distribución de módulos.
Respuesta:
* Se agrupa por paquete el cual es nombrado tomando como referencia el nombre del caso de uso 
* Los Fragment son la capa vista y no manejan logica de negocio, ni llamadas a servicios, se mantienen lo mas "tonta" posible. Es decir tiene que ser dependiente de la informacion y operaciones que realize la capa Presenter.
* Se establece la capa Presenter en base a contratos o especificaciones (interfaces) que describen las validaciones y operaciones que seran controladas a travez de esta capa, en esta misma se hace una especie de orquestacion de las funcionalidades donde se controla el inicio y fin, pero no se realiza logica de negocio, ni llamadas a ejecuciones en segundo plano ni mucho menos operaciones de networking.
* Implementando Clean Arquitecture la capa Model se dividen en:
    * La capa de datos donde establece la persistencia ya sea a una base de datos local tipo SQLite o Realm o uso de SharedPreferences, asi como tambien uso de servicios Web del tipo REST u otros usando Retrofit, Volley u otros para la persistencia en la nube. Ademas se puede aplicar el Repository Pattern el cual permite persistir localmente cuando se cuenta con una pobre o inexistente conexion a Internet o se usa una CACHE para eliminar la latencia tanto en el consumo de informacion como en la persistencia de datos. Si se usa el Repository Pattern la persistencia local es mas sencilla usando Room Library.
    * La capa de dominio donde se hace uso de Interactors o Use Cases, en la cual se ejecuta la logica de negocio y se controla la persistencia u obtencion de datos, asi como tambien se pueden disparar estas operaciones en segundo plano, gracias a esta capa se hace una diferenciacion y se añade una capa mas de abstraccion que permite desligar el modelo de datos de las operaciones. Esto permitira al Interactor realizar sus tareas sin responsabilizarse de la capa de datos.

4. Diseña un custom view de una brújula utilizando canvas y pon el código que utilizarías en esta sección.

```java
public class Brujula extends View {

    private static final Paint paint = new Paint(Paint.ANTI_ALIAS_FLAG);

    public Brujula(Context context) {
        super(context);
    }
    @Override
    protected void onDraw(Canvas canvas) {
        float grados = 45;//Norte apunta 45°

        int canvasWidth = canvas.getWidth();
        int canvasHeight = canvas.getHeight();

        int centroX = canvasWidth / 2;
        int centroY = canvasHeight / 2;
        
        int radio=Math.min(centroX,centroY);
        
        canvas.drawCircle(centroX, centroY, radio, paint);
        canvas.drawLine(centroX,centroY,(float) Math.cos(grados)*radio+centroX,(float) Math.sin(grados)*radio+centroY, paint);
        canvas.drawPaint( paint);

    }
}
```

