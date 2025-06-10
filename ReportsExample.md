# GNU Health – Ejemplos de Consultas SQL

## Estructura general:
- Las consultas se basan en datos médicos, demográficos y de laboratorio.
- Las fechas de interés suelen estar entre 2024-08-01 y 2025-01-31.
- Las principales tablas son: `gnuhealth_lab`, `gnuhealth_patient`, `gnuhealth_pathology`, `party_party`, etc.

---

## Ejemplo 1 – Casos de dengue por semana
**Pregunta del usuario:** Conteo de los casos de dengue por semana desde agosto del 2024 hasta enero del 2025.  
**SQL:**
```sql
SELECT 
  EXTRACT(YEAR FROM lab.create_date) AS Año,
  EXTRACT(WEEK FROM lab.create_date) AS SemanaEpidemiologica,
  COUNT(*) AS TotalCasos
FROM public.gnuhealth_lab AS lab
WHERE 
  lab.serializer LIKE '%Positivo%' AND 
  lab.serializer LIKE '%Dengue%' AND 
  lab.date_analysis BETWEEN '2024-08-01' AND '2025-01-31'
GROUP BY Año, SemanaEpidemiologica
ORDER BY Año, SemanaEpidemiologica ASC;
```

---

## Ejemplo 2 – Exámenes con resultados anormales
**Pregunta del usuario:** Lista de los exámenes de laboratorio ordenados por cantidad de resultados anormales.  
**SQL:**
```sql
SELECT
  DATE_TRUNC('month', lab.date_analysis) AS mes,
  test_type.name AS nombre_examen,
  COUNT(DISTINCT lab.patient) AS total_pacientes_anormales
FROM gnuhealth_lab AS lab
INNER JOIN gnuhealth_lab_test_critearea AS criteria ON lab.id = criteria.gnuhealth_lab_id
INNER JOIN gnuhealth_lab_test_type AS test_type ON lab.test = test_type.id
WHERE 
  lab.date_analysis BETWEEN '2024-08-01' AND '2025-01-31' AND
  (
    (criteria.result IS NOT NULL AND criteria.result >= criteria.lower_limit AND criteria.result <= criteria.upper_limit)
    OR
    (criteria.result IS NULL AND criteria.result_text ~ '^[0-9]+(\\.[0-9]+)?$' AND criteria.result_text::DOUBLE PRECISION >= criteria.lower_limit AND criteria.result_text::DOUBLE PRECISION <= criteria.upper_limit)
  )
GROUP BY mes, test_type.name
ORDER BY mes ASC, total_pacientes_anormales DESC;
```

---

## Ejemplo 3 – Diabetes y estado de control
**Pregunta del usuario:** Lista de pacientes con diabetes y su estado de control.  
**SQL:**
```sql
SELECT 
  d.name AS patient_id,
  CASE 
    WHEN lab.serializer IS NULL THEN 'Sin control' 
    ELSE lab.serializer 
  END AS casos_en_control
FROM gnuhealth_patient AS p
INNER JOIN gnuhealth_patient_disease AS d ON p.name = d.name
INNER JOIN gnuhealth_pathology AS pathology ON d.pathology = pathology.id
LEFT JOIN gnuhealth_lab AS lab ON p.name = lab.patient 
  AND lower(lab.serializer) LIKE '%glucosa%'  
  AND lower(lab.serializer) LIKE '%HP%'
WHERE lower(pathology.name) LIKE '%diabetes%'
ORDER BY d.name;
```

---

## Notas clave:
- `serializer` se usa para detectar resultados positivos o condiciones específicas.
- Las fechas clave se encuentran en `date_analysis`.
- Las relaciones entre pacientes y enfermedades están en `gnuhealth_patient_disease`.

---

**Instrucción para el LLM:**  
Cuando recibas una consulta del usuario sobre datos médicos o epidemiológicos, genera SQL basándote en estos ejemplos, usando las tablas relevantes. Considera fechas, condiciones médicas, tipo de datos y filtros lógicos como `LIKE`, `JOIN`, `BETWEEN`, `GROUP BY`, etc.