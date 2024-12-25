import mysql.connector
import streamlit as st
from dotenv import load_dotenv
import pandas as pd
import os

from localidades import ClaseLocalidad

load_dotenv()

class ClaseDireccion:
    def __init__(self) -> None:
        self.connection = mysql.connector.connect(
            host=os.getenv('DB_HOST'),
            user=os.getenv('DB_USER'),
            password=os.getenv('DB_PASSWORD'),
            database=os.getenv('DB_NAME'),
            port=os.getenv('DB_PORT'),
            ssl_disabled=False
        )
        self.cursor = self.connection.cursor(dictionary=True)

    def obtener_datos(self):
        self.cursor.execute('SELECT * FROM direccion')
        result = self.cursor.fetchall()
        return result
    
    def ingresar_datos(self, calle, numero, piso, departamento, localidad, latitud, longitud):
        self.cursor.execute('INSERT INTO direccion (calle_direccion, numero_direccion, piso_direccion, departamento_direccion, localidad_direccion, latitud_direccion, longitud_direccion) VALUES(%s,%s,%s,%s,%s,%s,%s)', (calle, numero, piso, departamento, localidad, latitud, longitud))
        self.connection.commit()
    
    def obtener_id_direccion(self, calle, numero):
        self.cursor.execute('SELECT id_direccion FROM direccion WHERE calle_direccion = %s AND numero_direccion = %s', (calle, numero))
        id = self.cursor.fetchall()
        return id

    def actualizar_datos(self, id, calle, numero, piso, departamento, localidad, latitud, longitud):
        consulta_actualizar = 'UPDATE direccion SET calle_direccion = %s, numero_direccion = %s, piso_direccion = %s, departamento_direccion = %s , localidad_direccion = %s, latitud_direccion = %s, longitud_direccion = %s WHERE id_direccion = %s '
        self.cursor.execute(consulta_actualizar, (calle, numero, piso, departamento, localidad, latitud, longitud, id))
        self.connection.commit()

class DataManagerDireccion:
    def __init__(self) -> None:
        self.db_direcciones = ClaseDireccion()
        
    def mostrar_datos(self):
        data_direcciones = self.db_direcciones.obtener_datos()
        return data_direcciones
    
    def agregar_registro(self):
        with st.container(border=True):
            col1, col2 = st.columns(2)
            
            with col1:
                calle = st.text_input("Calle")
                numero = st.text_input("Número")
                piso = st.text_input("Piso")
                departamento = st.text_input("Departamento")
                localidad = st.text_input("Localidad")
                latitud = st.text_input("Latitud")
                longitud = st.text_input("Longitud")
            
            if st.button("Agregar Dirección"):
                try:
                    self.db_direcciones.ingresar_datos(calle, numero, piso, departamento, localidad, latitud, longitud)
                    st.success('Dirección agregada correctamente.')
                except Exception as e:
                    st.error(f'Error al agregar dirección: {e}')
    
    def editar_registro(self, id):
        with st.container(border=True):
            col1, col2 = st.columns(2)
            
            with col1:
                calle = st.text_input("Calle")
                numero = st.text_input("Número")
                piso = st.text_input("Piso")
                departamento = st.text_input("Departamento")
                localidad = st.text_input("Localidad")
                latitud = st.text_input("Latitud")
                longitud = st.text_input("Longitud")
            
            if st.button("Actualizar Dirección"):
                try:
                    self.db_direcciones.actualizar_datos(id, calle, numero, piso, departamento, localidad, latitud, longitud)
                    st.success('Dirección actualizada correctamente.')
                except Exception as e:
                    st.error(f'Error al actualizar dirección: {e}')
    
    def eliminar_registro(self):
        data_direcciones = pd.DataFrame(self.db_direcciones.obtener_datos())
        selected_id = st.selectbox("Seleccionar dirección para eliminar:", data_direcciones['id_direccion'].tolist())
        
        if st.button("Eliminar Dirección"):
            try:
                self.db_direcciones.cursor.execute('DELETE FROM direccion WHERE id_direccion = %s', (selected_id,))
                self.db_direcciones.connection.commit()
                st.success('Dirección eliminada correctamente.')
            except Exception as e:
                st.error(f'Error al eliminar dirección: {e}')

                
