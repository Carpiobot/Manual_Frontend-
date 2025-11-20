#Manual Técnico Frontend: Test de Compras compulsivas

#Manual de Creación: Aplicación de Test de Compras Compulsivas
#Esta guía detalla el proceso de construcción de una aplicación web para evaluar y proporcionar información sobre hábitos de compra, utilizando React, Tailwind CSS y la plataforma Base44.
#Objetivo de la Aplicación: Evaluar los hábitos de compra de los usuarios mediante un test, ofrecer un análisis de riesgo de compras compulsivas, proporcionar recomendaciones personalizadas y permitir el seguimiento del historial de resultados, además de contar con un psicólogo virtual basado en IA.
#Paso 1: Diseño de Entidades (Base de Datos)
La piedra angular de la aplicación es la entidad TestResult, que almacena los resultados de cada test realizado por el usuario.
#Entidad: TestResult.json
#Propósito: Guardar el resultado completo de cada test.
#Atributos clave (schema):
#score (number): Puntuación total del test.
#risk_level (string: "bajo", "moderado", "alto", "muy_alto"): Nivel de riesgo calculado.
#personal_info (object): Información demográfica del usuario (carrera, semestre, edad, género, trabaja).
#answers (array of objects): Detalles de cada pregunta respondida (ID de pregunta, respuesta, texto de la respuesta).
#recommendations (array of strings): Recomendaciones personalizadas generadas por la IA o predefinidas.
#completion_time (number): Tiempo en segundos que tomó completar el test.
#ai_insights (string): Análisis general generado por la IA.
#Cómo se define: Se crea un archivo entities/TestResult.json con este esquema.
#Uso: La aplicación interactúa con esta entidad usando base44.entities.TestResult.create(), list(), etc.

#Paso 2: Estructura del Frontend (Páginas y Componentes)
#La interfaz de usuario se construye con React, organizada en páginas principales y componentes reutilizables.
#A. Páginas Principales (pages/)
#Test.js (Página de Inicio/Test):
#Propósito: Guiar al usuario a través del test de compras compulsivas.
#Flujo:
#TestIntro: Pantalla de bienvenida y descripción del test.
#TestPersonalInfo: Recopilación de información personal (carrera, edad, etc.).
#TestQuestion: Muestra las preguntas del test una por una.
#TestResults: Muestra los resultados finales, análisis y recomendaciones.
#Lógica: Gestiona el estado del test (pregunta actual, respuestas, información personal, tiempo), calcula la puntuación, determina el nivel de riesgo y, al finalizar, guarda los resultados en la entidad TestResult e invoca a la IA para análisis.
#Interacción con IA: Utiliza base44.integrations.Core.InvokeLLM para obtener ai_insights y recommendations avanzados.
#History.js (Página de Historial):
#Propósito: Mostrar el historial de tests realizados por el usuario.
#Contenido:
#Resumen del último test y métricas generales (promedio, tendencia).
#HistoryChart: Un gráfico de líneas que visualiza la evolución de las puntuaciones a lo largo del tiempo.
#TestCard: Una lista de todas las TestResult del usuario, cada una expandible para ver detalles (información personal, respuestas, análisis de IA y recomendaciones).
#Lógica: Carga todos los TestResult del usuario ordenados por fecha (base44.entities.TestResult.list("-created_date")).
#Psicologo.js (Página de Psicólogo Virtual):
#Propósito: Proporcionar una interfaz de chat con un agente de IA especializado.
#Funcionalidad:
#Lista de conversaciones existentes.
#Interfaz de chat para interactuar con el agente psicologo_compras.
#Manejo del estado de la conversación y envío/recepción de mensajes.
#Interacción con Agentes: Utiliza el SDK de agentes de Base44 (base44.agents.listConversations, getConversation, addMessage, subscribeToConversation).
#Info.js (Página de Información):
#Propósito: Educar a los usuarios sobre las compras compulsivas.
#Contenido: Información textual sobre el trastorno, síntomas, cómo afecta, consejos y recursos.
#B. Componentes Reutilizables (components/)
#test/TestIntro.js: Pantalla de bienvenida del test.
#test/TestPersonalInfo.js: Formulario para recopilar datos demográficos.
#test/TestQuestion.js: Componente para mostrar una pregunta y sus opciones de respuesta.
#test/TestResults.js: Muestra la puntuación, nivel de riesgo y recomendaciones finales.
#history/HistoryChart.js: Componente de gráfico (usando Recharts) para visualizar la evolución de puntuaciones.
#history/TestCard.js: Tarjeta expandible para mostrar los detalles de un test individual en el historial.
#chat/MessageBubble.js: Muestra un mensaje en el chat, diferenciando entre usuario y IA, y renderizando resultados de tool calls de la IA.
#chat/ConversationList.js: Lista las conversaciones de chat del usuario.
#ui/sidebar y otros componentes Shadcn/UI: Utilizados para construir la interfaz de usuario con un diseño coherente.
#Paso 3: Layout de la Aplicación (Layout.js)
#Propósito: Proporcionar una estructura de navegación y un diseño visual consistente para toda la aplicación.
#Elementos:
#Sidebar: Menú de navegación lateral con enlaces a las páginas principales (Test, History, Psicologo, Info). Utiliza <Link to={createPageUrl(pageName)}> para la #navegación interna.
#Header (para móvil): Encabezado superior con botón para abrir la Sidebar.
#Área de contenido: Donde se renderiza el children (la página actual).
#Estilo: Definición de variables CSS personalizadas y uso de gradientes y sombras para una estética moderna.
#Paso 4: Integración de Inteligencia Artificial
#Análisis de Test (base44.integrations.Core.InvokeLLM):
#En pages/Test.js, al finalizar el test, se invoca a Core.InvokeLLM.
#Prompt Detallado: Se construye un prompt que incluye la puntuación, las respuestas a cada pregunta y la información personal del usuario. Se le pide a la IA que actúe como un psicólogo especializado.
#response_json_schema: Se especifica un esquema JSON para que la IA devuelva el análisis general (analisis_general) y una lista de recomendaciones (recomendaciones) en un formato estructurado. Esto garantiza que la aplicación pueda parsear y mostrar la respuesta de la IA correctamente.
#Agente Psicólogo Virtual (agents/psicologo_compras.json):
#Propósito: Crear una IA conversacional especializada en el tema.
#Definición:
#description e instructions: Describen el rol del agente (psicólogo especializado en compras compulsivas).
#tool_configs: Permite que el agente interactúe con entidades, por ejemplo, {"entity_name": "TestResult", "allowed_operations": ["read"]} para que pueda leer el historial de tests del usuario si es relevante para la conversación.
#whatsapp_greeting: Un mensaje de bienvenida para la integración con WhatsApp.
#Uso en Frontend: La página Psicologo.js utiliza el SDK de agentes de Base44 para gestionar las conversaciones con este agente.
#Paso 5: Estilizado y Diseño Responsivo
#Tailwind CSS: Utilizado para todos los estilos, permitiendo un desarrollo rápido y un diseño consistente.
#Clases Utilitarias: Clases como flex, grid, p-4, text-xl, bg-gradient-to-br se utilizan extensivamente para crear diseños flexibles y visualmente atractivos.
#motion (Framer Motion): Añade animaciones suaves y transiciones entre los pasos del test y al expandir elementos en el historial, mejorando la experiencia de usuario.
#Componentes Shadcn/UI: Proporcionan componentes pre-estilizados y accesibles (Card, Button, Progress, Badge) que se integran perfectamente con Tailwind.
#Responsividad: El diseño se planifica para adaptarse a diferentes tamaños de pantalla utilizando clases responsivas de Tailwind (md:, lg:).
#Paso 6: Utilidades
#createPageUrl(pageName): Una función utilitaria que genera URL seguras para la navegación entre páginas de la aplicación.
#date-fns: Utilizado para formatear fechas en el historial (format(new Date(result.created_date), 'PPP', { locale: es })).
#Consideraciones Adicionales:
#Manejo de Carga: Se implementan estados de carga (isLoading) para proporcionar retroalimentación visual al usuario mientras se realizan operaciones asíncronas (ej. carga del historial, finalización del test con IA).
#Mensajes de Vacío: Cuando no hay historial, se muestra un mensaje amigable con una llamada a la acción.
#Validación de Formulario: En Test.js, la navegación al siguiente paso de preguntas solo se habilita si la pregunta actual ha sido respondida (isCurrentQuestionAnswered()).
#Integración de Logotipos: Uso de mixBlendMode: 'multiply' para que los logotipos con fondo blanco se integren mejor con fondos de color, como se implementó recientemente en TestIntro.js.
Este manual cubre los aspectos fundamentales para replicar la aplicación. Cada componente y página fue diseñado con un propósito específico, contribuyendo a la funcionalidad general y la experiencia del usuario.
