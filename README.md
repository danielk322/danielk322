import mysql.connector
import streamlit as st
from dotenv import load_dotenv
import pandas as pd
import os

# Cargar variables de entorno desde el archivo .env
load_dotenv()

class ClaseLocalidad:
    def __init__(self) -> None:
        # Establecer la conexión con la base de datos
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
        """Obtiene todos los datos de la tabla 'localidad'."""
        self.cursor.execute('SELECT * FROM localidad')
        return self.cursor.fetchall()

class Localidad:
    def __init__(self, nombre_localidad: str, provincia: str):
        self.nombre_localidad = nombre_localidad
        self.provincia = provincia

    def __str__(self) -> str:
        return f"{self.nombre_localidad}, {self.provincia}"

    def __repr__(self) -> str:
        return f"Localidad(nombre_localidad='{self.nombre_localidad}', provincia='{self.provincia}')"

class Direccion:
    def __init__(self, calle: str, numero: int, departamento: str = None, 
                 piso: int = None, localidad: Localidad = None,
                 latitud: float = None, longitud: float = None):
        self.calle = calle
        self.numero = numero
        self.departamento = departamento
        self.piso = piso
        self.localidad = localidad
        self.latitud = latitud
        self.longitud = longitud

    def direccion_completa(self) -> str:
        """Genera la dirección completa como cadena."""
        direccion = f"{self.calle} {self.numero}"
        if self.piso is not None:
            direccion += f", Piso {self.piso}"
        if self.departamento is not None:
            direccion += f", Depto {self.departamento}"
        if self.localidad is not None:
            direccion += f", {str(self.localidad)}"
        return direccion

    def tiene_coordenadas(self) -> bool:
        """Indica si la dirección tiene coordenadas de latitud y longitud."""
        return self.latitud is not None and self.longitud is not None

    def __str__(self) -> str:
        return self.direccion_completa()

    def __repr__(self) -> str:
        return (f"Direccion(calle='{self.calle}', numero={self.numero}, "
                f"departamento='{self.departamento}', piso={self.piso}, "
                f"localidad={repr(self.localidad)}, latitud={self.latitud}, "
                f"longitud={self.longitud})")

                
