import mysql.connector  # Importa la librería para conectarse a MySQL
import streamlit as st  # Importa Streamlit para crear la interfaz de usuario
from dotenv import load_dotenv  # Importa dotenv para gestionar variables de entorno
import pandas as pd  # Importa Pandas para manejo de datos en DataFrames
import os  # Importa el módulo os para acceder a las variables de entorno

# Importa la clase ClaseLocalidad desde un módulo llamado localidades
from localidades import ClaseLocalidad

load_dotenv()  # Carga las variables de entorno desde el archivo .env

# Definición de la clase ClaseDireccion
class ClaseDireccion:
    def __init__(self) -> None:
        # Inicializa la conexión a la base de datos MySQL utilizando las variables de entorno
        self.connection = mysql.connector.connect(
            host=os.getenv('DB_HOST'),  # Obtiene la dirección del host de la base de datos
            user=os.getenv('DB_USER'),  # Obtiene el nombre de usuario para la conexión
            password=os.getenv('DB_PASSWORD'),  # Obtiene la contraseña para la conexión
            database=os.getenv('DB_NAME'),  # Obtiene el nombre de la base de datos
            port=os.getenv('DB_PORT'),  # Obtiene el puerto para la conexión
            ssl_disabled=False  # Configura SSL si es necesario (en este caso, desactivado)
        )
        self.cursor = self.connection.cursor(dictionary=True)  # Crea un cursor con resultados en diccionario

    # Método para obtener todos los registros de la tabla 'direccion'
    def obtener_datos(self):
        self.cursor.execute('SELECT * FROM direccion')  # Ejecuta una consulta SQL para obtener todas las direcciones
        result = self.cursor.fetchall()  # Recupera todos los resultados como una lista de diccionarios
        return result  # Retorna los datos obtenidos

    # Método para agregar una nueva dirección a la base de datos
    def ingresar_datos(self, calle, numero, piso, departamento, localidad, latitud, longitud):
        # Inserta una nueva dirección en la tabla 'direccion'
        self.cursor.execute(
            'INSERT INTO direccion (calle_direccion, numero_direccion, piso_direccion, departamento_direccion, localidad_direccion, latitud_direccion, longitud_direccion) VALUES(%s,%s,%s,%s,%s,%s,%s)',
            (calle, numero, piso, departamento, localidad, latitud, longitud)
        )
        self.connection.commit()  # Confirma los cambios en la base de datos

    # Método para obtener el ID de una dirección específica por calle y número
    def obtener_id_direccion(self, calle, numero):
        self.cursor.execute(
            'SELECT id_direccion FROM direccion WHERE calle_direccion = %s AND numero_direccion = %s',
            (calle, numero)
        )
        id = self.cursor.fetchall()  # Recupera el ID correspondiente
        return id  # Retorna el ID obtenido

    # Método para actualizar los datos de una dirección existente
    def actualizar_datos(self, id, calle, numero, piso, departamento, localidad, latitud, longitud):
        consulta_actualizar = 'UPDATE direccion SET calle_direccion = %s, numero_direccion = %s, piso_direccion = %s, departamento_direccion = %s , localidad_direccion = %s, latitud_direccion = %s, longitud_direccion = %s WHERE id_direccion = %s '
        self.cursor.execute(
            consulta_actualizar,
            (calle, numero, piso, departamento, localidad, latitud, longitud, id)
        )
        self.connection.commit()  # Confirma los cambios en la base de datos

# Clase para gestionar las operaciones relacionadas con direcciones
class DataManagerDireccion:
    def __init__(self) -> None:
        self.db_direcciones = ClaseDireccion()  # Inicializa la clase ClaseDireccion para gestión de direcciones

    # Método para mostrar todos los registros de direcciones
    def mostrar_datos(self):
        data_direcciones = self.db_direcciones.obtener_datos()  # Obtiene los datos de la clase ClaseDireccion
        return data_direcciones  # Retorna los datos

    # Método para agregar una nueva dirección
    def agregar_registro(self):
        with st.container(border=True):  # Crea un contenedor en Streamlit con borde
            col1, col2 = st.columns(2)  # Divide la interfaz en dos columnas

            with col1:
                # Captura los datos de la nueva dirección a través de inputs en Streamlit
                calle = st.text_input("Calle")  
                numero = st.text_input("Número")
                piso = st.text_input("Piso")
                departamento = st.text_input("Departamento")
                localidad = st.text_input("Localidad")
                latitud = st.text_input("Latitud")
                longitud = st.text_input("Longitud")

            # Botón para agregar la nueva dirección
            if st.button("Agregar Dirección"):
                try:
                    self.db_direcciones.ingresar_datos(calle, numero, piso, departamento, localidad, latitud, longitud)
                    st.success('Dirección agregada correctamente.')  # Mensaje de éxito
                except Exception as e:
                    st.error(f'Error al agregar dirección: {e}')  # Mensaje de error si falla

    # Método para editar una dirección existente
    def editar_registro(self, id):
        with st.container(border=True):  # Crea un contenedor en Streamlit con borde
            col1, col2 = st.columns(2)  # Divide la interfaz en dos columnas

            with col1:
                # Captura los nuevos datos para la dirección a través de inputs en Streamlit
                calle = st.text_input("Calle")  
                numero = st.text_input("Número")
                piso = st.text_input("Piso")
                departamento = st.text_input("Departamento")
                localidad = st.text_input("Localidad")
                latitud = st.text_input("Latitud")
                longitud = st.text_input("Longitud")

            # Botón para actualizar los datos de la dirección
            if st.button("Actualizar Dirección"):
                try:
                    self.db_direcciones.actualizar_datos(id, calle, numero, piso, departamento, localidad, latitud, longitud)
                    st.success('Dirección actualizada correctamente.')  # Mensaje de éxito
                except Exception as e:
                    st.error(f'Error al actualizar dirección: {e}')  # Mensaje de error si falla

    # Método para eliminar una dirección existente
    def eliminar_registro(self):
        data_direcciones = pd.DataFrame(self.db_direcciones.obtener_datos())  # Obtiene los datos de las direcciones
        selected_id = st.selectbox("Seleccionar dirección para eliminar:", data_direcciones['id_direccion'].tolist())  # Selecciona una dirección por su ID

        # Botón para eliminar la dirección seleccionada
        if st.button("Eliminar Dirección"):
            try:
                self.db_direcciones.cursor.execute('DELETE FROM direccion WHERE id_direccion = %s', (selected_id,))
                self.db_direcciones.connection.commit()  # Confirma los cambios en la base de datos
                st.success('Dirección eliminada correctamente.')  # Mensaje de éxito
            except Exception as e:
                st.error(f'Error al eliminar dirección: {e}')  # Mensaje de error si falla

                
