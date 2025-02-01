
# Proyecto de Predicción de Probabilidades de avistamiento con Flutter y Flask

Este proyecto tiene como objetivo crear una aplicación móvil que realice predicciones sobre probabilidades de avistamiento en diferentes zonas usando un modelo de aprendizaje automático entrenado con un conjunto de datos. El backend está implementado en Flask con un modelo Random Forest y el frontend se desarrollará con Flutter.


# Tecnologías Utilizadas
- Backend: Flask, Random Forest (scikit-learn), 
- Pandas
- Frontend: Flutter (para la aplicación móvil)
- Modelo de Machine Learning: Random Forest Regressor
- Base de Datos: No se requiere una base de datos, los datos se procesan directamente desde un   archivo CSV proporcionado.
- API: RESTful API para comunicación entre el frontend y el backend.

# Requisitos del Proyecto
1. Backend (Flask)
2. Python 3.10 o superior


# Instalar las dependencias necesarias:






```bash
  pip install Flask pandas scikit-learn

```
    
## Frontend (Flutter)

Flutter 3.0 o superior
Instalar dependencias de Flutter:

### Adicionar al archivo yaml
```bash
dependencies:
  flutter:
    sdk: flutter

  cupertino_icons: ^1.0.6
  dio: ^5.8.0+1
  go_router: ^7.0.1
  riverpod: ^2.5.1
  flutter_riverpod: ^2.5.1
  riverpod_annotation: ^2.3.5
  lottie: ^2.4.0
```
### Ejecutar
```bash
flutter pub get
```

# Configuración del Backend
## Pasos para configurar el servidor Flask en el Backend
1. Instalar las dependencias necesarias:

- Flask para la API
- Pandas y scikit-learn para el procesamiento y el modelo de predicción.
Ejemplo:

``` bash
pip install flask pandas scikit-learn
```

2. Implementar la API con Flask: Aquí se describe el código que carga un conjunto de datos CSV, entrena un modelo Random Forest, realiza predicciones y devuelve los resultados en formato JSON.

``` bash
import pandas as pd
import json
from flask import Flask, jsonify
from sklearn.model_selection import train_test_split
from sklearn.ensemble import RandomForestRegressor
from sklearn.preprocessing import OneHotEncoder

app = Flask(__name__)

@app.route('/probabilidad_caza', methods=['GET'])
def obtener_probabilidad_caza():
    url = "https://data-cdfw.opendata.arcgis.com/api/download/v1/items/c5a5fcce39cf44f5b4f218781cbd2bb1/csv?layers=0"
    df = pd.read_csv(url)

    df = df[['Arcreage', 'Shape__Length', 'MapLabel']]
    df = df.dropna()
    df['Densidad_Poblacion'] = df['Arcreage'] / df['Shape__Length']

    encoder = OneHotEncoder(sparse_output=False)
    maplabel_encoded = encoder.fit_transform(df[['MapLabel']])
    maplabel_df = pd.DataFrame(maplabel_encoded, columns=encoder.get_feature_names_out(['MapLabel']))
    df = pd.concat([df, maplabel_df], axis=1)

    X = df.drop(['Arcreage', 'MapLabel'], axis=1)
    y = df['Arcreage']

    X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

    modelo = RandomForestRegressor(n_estimators=100, random_state=42)
    modelo.fit(X_train, y_train)

    df['Prediccion'] = modelo.predict(X)

    predicciones_por_zona = df.groupby('MapLabel')['Prediccion'].mean()
    total_prediccion = predicciones_por_zona.sum()
    porcentaje_por_zona = (predicciones_por_zona / total_prediccion) * 100

    zonas_porcentaje_listado = [
        {"Zona": zona, "Porcentaje_Probabilidad_Caza": round(porcentaje, 2)}
        for zona, porcentaje in porcentaje_por_zona.items()
    ]

    response = {"response": zonas_porcentaje_listado}
    return jsonify(response)

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8720)

```

