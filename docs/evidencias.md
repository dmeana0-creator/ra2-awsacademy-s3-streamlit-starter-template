# Evidencias · RA2 SBD (rellenar por el alumnado)

> Completa este documento con capturas/salidas. No incluyas secretos.
> Indica si has usado **Variante A (IAM Role)** o **Variante B (aws configure)**.

## 0) Identificación
- Alumno/a: Diego
- Grupo: IABD
- Variante usada (A/B): B
- Región AWS: us-east-1
- Bucket S3: iabd03-Tarea-RA2

---

## 1) S3 privado
- [x] Captura del bucket (nombre y región)
![Captura nombre y región del bucket](../img/nombre_y_region_bucket.png)

- [x] Captura/confirmación de que **no es público** (Block Public Access o permisos)
![Captura privacidad bucket](../img/privacidad_bucket.png)

- [x] Captura del objeto JSON en `data/sensores/`
![Captura JSON en el bucket](../img/json_en_datasensores.png)


**Notas:**
- Key usada (S3_KEY): `s3://iabd03-tarea-ra2/data/sensores/iabd03_sensores.json`

---

## 2) Notebook / Script de subida
- [x] Captura de la ejecución del notebook/script subiendo a S3
![Captura 1 del notebook de subida a S3](../img/notebook_celda_1.png)
![Captura 2 del notebook de subida a S3](../img/notebook_celda_2.png)


- [x] Enlace o ruta del archivo en el repo (`notebooks/...`): `notebooks/ingesta_json_S3.ipynb`

---

## 3) EC2 y red
- [x] Captura de la instancia EC2 (Ubuntu 22.04)
![Captura 1 de la instancia EC2](../img/ec2_captura_1.png)
![Captura 2 de la instancia EC2](../img/ec2_captura_2.png)


- [x] Captura del Security Group con puerto 8501 abierto (según reglas del lab)
![Captura del Security Group con puerto 8501 abierto](../img/grupo_seguridad.png)

- [x] Salida de `ssh` conectando (sin mostrar claves)
![Captura Salida de `ssh` conectando (sin mostrar claves)](../img/conexión_ssh.png)


---

## 4) Acceso a S3 desde EC2 (sin secretos)
Ejecuta en EC2:

```bash
aws sts get-caller-identity
aws s3 ls s3://iabd03-tarea-ra2/data/sensores/
```

- [x] Captura/salida de ambos comandos
![Captura salida de comandos aws](../img/comandos_aws.png)

---

## 5) Streamlit en EC2
- [x] Captura de `streamlit hello` funcionando (o `python -c "import streamlit"`)
![Captura 1 streamlit hello funcionando](../img/hello_captura_1.png)
![Captura 2 streamlit hello funcionando](../img/hello_captura_2.png)

- [x] Captura de instalación de dependencias (`pip install -r requirements.txt`)
![Captura instalación de dependencias](../img/dependencias_instaladas.png)

---

## 6) Dashboard (funcionalidad)
Incluye capturas donde se vea:

- [ ] Filtro por `sensor_state`
![Captura filtro por sensor_state]()

- [ ] Slider de temperatura
![Captura slider de temperatura]()

- [ ] Tabla filtrada
![Captura tabla filtrada]()

- [ ] Gráfica línea (temperatura vs tiempo)
![Captura gráfica línea temperatura vs tiempo]()

- [ ] Gráfica barras (CO₂ por sensor)
![Captura gráfica barras (CO₂ por sensor)]()

- [ ] Mapa con sensores
![Captura mapa con sensores]()

---

## 7) Despliegue final
- [ ] Comando usado para arrancar en segundo plano (ej. `nohup` o script): 

- [ ] Captura del log (`tail -n 50 streamlit.log` o similar)
![Captura del log]()

- [ ] URL final: 

**URL:** `http://IP_PUBLICA_EC2:8501`

- [ ] Captura en navegador accediendo a la URL
![Captura navegador accediendo a la URL]()

---

## 8) Observaciones (opcional)
- Problemas encontrados y solución:
