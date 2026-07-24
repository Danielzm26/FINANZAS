import streamlit as st
import pandas as pd
import yfinance as yf
from datetime import datetime
import os
import plotly.express as px
import plotly.graph_objects as go


# =========================
# CONFIGURACION
# =========================

st.set_page_config(
    page_title="Mi Portafolio",
    page_icon="📈",
    layout="wide"
)


FILE = "portfolio.csv"
CAPITAL_FILE = "capital.csv"
FINANZAS_FILE = "finanzas.csv"



# =========================
# CARGAR OPERACIONES
# =========================

if os.path.exists(FILE):

    df = pd.read_csv(FILE)


else:

    df = pd.DataFrame(
        columns=[
            "Fecha",
            "Ticker",
            "Tipo",
            "Precio_Compra",
            "Cantidad"
        ]
    )



# =========================
# CARGAR CAPITAL
# =========================

if os.path.exists(CAPITAL_FILE):

    capital_df = pd.read_csv(
        CAPITAL_FILE
    )


else:

    capital_df = pd.DataFrame(
        columns=[
            "Fecha",
            "Capital"
        ]
    )




# =========================
# TITULO
# =========================

st.title(
    "📊 Control Profesional de Portafolio"
)




# =========================
# CAPITAL
# =========================

st.sidebar.header(
    "💰 Gestión de Capital"
)



capital_inicial = st.sidebar.number_input(

    "Capital inicial",

    value=10000.0,

    step=100.0

)



# Crear capital inicial solamente la primera vez

if capital_df.empty:


    nuevo_capital = pd.DataFrame(

        [[

            datetime.now(),

            capital_inicial

        ]],

        columns=[

            "Fecha",

            "Capital"

        ]

    )


    capital_df = pd.concat(

        [

            capital_df,

            nuevo_capital

        ],

        ignore_index=True

    )


    capital_df.to_csv(

        CAPITAL_FILE,

        index=False

    )





# =========================
# AGREGAR CAPITAL
# =========================

st.sidebar.subheader(
    "➕ Añadir dinero"
)



nuevo_deposito = st.sidebar.number_input(

    "Nuevo capital agregado",

    value=0.0,

    step=100.0

)




if st.sidebar.button(
    "💰 Guardar capital"
):


    if nuevo_deposito > 0:


        nuevo = pd.DataFrame(

            [[

                datetime.now(),

                nuevo_deposito

            ]],

            columns=[

                "Fecha",

                "Capital"

            ]

        )



        capital_df = pd.concat(

            [

                capital_df,

                nuevo

            ],

            ignore_index=True

        )



        capital_df.to_csv(

            CAPITAL_FILE,

            index=False

        )



        st.sidebar.success(

            "Capital agregado correctamente"

        )


        st.rerun()





capital_total = capital_df["Capital"].sum()




# =========================
# NUEVA OPERACION
# =========================


st.subheader(
    "➕ Registrar operación"
)




c1,c2,c3,c4 = st.columns(4)




with c1:

    ticker = st.text_input(

        "Ticker",

        "NVDA"

    )



with c2:

    tipo = st.selectbox(

        "Operación",

        [

            "BUY",

            "SELL"

        ]

    )



with c3:

    precio_compra = st.number_input(

        "Precio",

        value=100.0,

        step=0.10

    )



with c4:

    cantidad = st.number_input(

        "Cantidad",

        value=1,

        step=1

    )





if st.button(
    "💾 Registrar operación"
):


    nueva_operacion = pd.DataFrame(

        [[

            datetime.now(),

            ticker.upper(),

            tipo,

            precio_compra,

            cantidad

        ]],

        columns=df.columns

    )



    df = pd.concat(

        [

            df,

            nueva_operacion

        ],

        ignore_index=True

    )



    df.to_csv(

        FILE,

        index=False

    )


    st.success(

        "Operación guardada"

    )


    st.rerun()




# =========================
# FIN PARTE 1
# =========================
# =========================
# CALCULO DEL PORTAFOLIO
# =========================


portfolio = {}


# Dinero disponible actual

cash = capital_total



for _, row in df.iterrows():


    ticker = row["Ticker"]

    cantidad = float(row["Cantidad"])

    precio = float(row["Precio_Compra"])


    valor_operacion = cantidad * precio



    if ticker not in portfolio:


        portfolio[ticker] = {

            "cantidad": 0,

            "costo": 0,

            "vendido": 0

        }




    # =====================
    # COMPRA
    # =====================


    if row["Tipo"] == "BUY":



        portfolio[ticker]["cantidad"] += cantidad


        portfolio[ticker]["costo"] += valor_operacion


        # Sale dinero del efectivo

        cash -= valor_operacion





    # =====================
    # VENTA
    # =====================


    elif row["Tipo"] == "SELL":



        portfolio[ticker]["cantidad"] -= cantidad


        portfolio[ticker]["vendido"] += valor_operacion


        # Entra dinero al efectivo

        cash += valor_operacion





# Evitar valores negativos pequeños

if cash < 0 and cash > -0.01:

    cash = 0




# =========================
# CREAR TABLA ACTUAL
# =========================


datos = []



for ticker, info in portfolio.items():



    cantidad = info["cantidad"]



    if cantidad <= 0:

        continue



    costo_promedio = (

        info["costo"] / cantidad

    )



    try:


        precio_actual = float(

            yf.Ticker(ticker)

            .history(

                period="1d",

                auto_adjust=False

            )

            ["Close"]

            .iloc[-1]

        )


    except:


        precio_actual = 0




    valor_actual = cantidad * precio_actual



    ganancia = (

        valor_actual -

        info["costo"]

    )



    ganancia_pct = (


        (precio_actual / costo_promedio) - 1

    ) * 100

    datos.append([

        ticker,

        cantidad,

        info["costo"],

        costo_promedio,

        precio_actual,

        valor_actual,

        ganancia,

        ganancia_pct

    ])




df_port = pd.DataFrame(

    datos,

    columns=[

        "Ticker",

        "Cantidad",

        "Costo Total",

        "Precio Promedio Compra",

        "Precio Actual",

        "Valor Actual",

        "Ganancia $",

        "Ganancia %"

    ]

)




# =========================
# METRICAS PRINCIPALES
# =========================


valor_acciones = (

    df_port["Valor Actual"].sum()

    if not df_port.empty

    else 0

)



valor_total = (

    cash + valor_acciones

)



ganancia_total = (

    valor_total - capital_total

)



rendimiento_pct = (


    (ganancia_total / capital_total) * 100


    if capital_total > 0


    else 0

)





# =========================
# DASHBOARD FINANCIERO
# =========================


st.subheader(
    "💰 Estado actual del dinero"
)



m1,m2,m3,m4 = st.columns(4)



m1.metric(

    "💵 Dinero disponible",

    f"${cash:,.2f}"

)



m2.metric(

    "📈 Valor acciones",

    f"${valor_acciones:,.2f}"

)



m3.metric(

    "🏦 Patrimonio total",

    f"${valor_total:,.2f}"

)



m4.metric(

    "🚀 Rendimiento",

    f"{rendimiento_pct:.2f}%"

)





# =========================
# RESUMEN DETALLADO
# =========================


st.subheader(
    "📊 Resumen financiero"
)



resumen = pd.DataFrame(

    {

        "Concepto":[

            "Capital aportado",

            "Dinero invertido",

            "Cash disponible",

            "Valor acciones",

            "Valor total",

            "Ganancia/Pérdida"

        ],


        "Monto":[

            capital_total,

            valor_acciones,

            cash,

            valor_acciones,

            valor_total,

            ganancia_total

        ]

    }

)



st.dataframe(

    resumen,

    use_container_width=True

)




# =========================
# TABLA PORTAFOLIO
# =========================


st.subheader(
    "📌 Mis posiciones"
)



if not df_port.empty:


    st.dataframe(

        df_port,

        use_container_width=True

    )




# =========================
# FIN PARTE 2
# =========================
# =========================
# DISTRIBUCION DEL CAPITAL
# =========================


st.subheader(
    "🥧 Distribución del capital"
)



if not df_port.empty:


    df_pie = df_port[

        [

            "Ticker",

            "Valor Actual"

        ]

    ].copy()



    df_pie.rename(

        columns={

            "Valor Actual":"Valor"

        },

        inplace=True

    )



    df_cash = pd.DataFrame(

        {

            "Ticker":[

                "Cash"

            ],

            "Valor":[

                cash

            ]

        }

    )



    df_pie = pd.concat(

        [

            df_pie,

            df_cash

        ],

        ignore_index=True

    )



    fig_pie = px.pie(

        df_pie,

        values="Valor",

        names="Ticker",

        hole=0.45

    )



    st.plotly_chart(

        fig_pie,

        use_container_width=True

    )





# =========================
# GRAFICA HISTORICA
# =========================


st.subheader(
    "📈 Historial de precio y compras"
)



if not df_port.empty:


    seleccion = st.selectbox(

        "Selecciona acción",

        df_port["Ticker"]

    )



    try:



        historial = yf.Ticker(

            seleccion

        ).history(

            period="1y",

            auto_adjust=False

        )



        fig = go.Figure()



        fig.add_trace(

            go.Scatter(

                x=historial.index,

                y=historial["Close"],

                mode="lines",

                name="Precio"

            )

        )

        precio_promedio = df_port[
            df_port["Ticker"] == seleccion
            ]["Precio Promedio Compra"].iloc[0]



        fig.add_hline(

            y=precio_promedio,

            annotation_text="Precio promedio compra"

        )



        # Normalizar fechas

        historial_index = historial.index



        if historial_index.tz is not None:


            historial_index = historial_index.tz_localize(None)





        compras = df[

            (df["Ticker"] == seleccion)

            &

            (df["Tipo"] == "BUY")

        ]





        for _, compra in compras.iterrows():



            fecha = pd.to_datetime(

                compra["Fecha"]

            )



            if fecha.tzinfo is not None:


                fecha = fecha.tz_localize(None)



            posicion = historial_index.get_indexer(

                [fecha],

                method="nearest"

            )



            if posicion[0] >= 0:



                fecha_real = historial_index[

                    posicion[0]

                ]



                fig.add_trace(

                    go.Scatter(

                        x=[fecha_real],

                        y=[compra["Precio_Compra"]],

                        mode="markers",

                        marker=dict(

                            size=15,

                            symbol="triangle-up"

                        ),

                        name="Compra"

                    )

                )




        fig.update_layout(

            title=f"{seleccion} - Evolución",

            height=550

        )



        st.plotly_chart(

            fig,

            use_container_width=True

        )



    except Exception as e:



        st.warning(

            f"Error cargando gráfica: {e}"

        )





# =========================
# HISTORIAL OPERACIONES
# =========================


st.subheader(
    "📜 Historial de operaciones"
)



st.dataframe(

    df,

    use_container_width=True

)





# =========================
# HISTORIAL DE CAPITAL
# =========================


st.subheader(
    "💰 Historial de aportaciones"
)



st.dataframe(

    capital_df,

    use_container_width=True

)





# =========================
# EVOLUCION DEL CAPITAL
# =========================


st.subheader(
    "📈 Evolución de aportaciones"
)



if not capital_df.empty:


    capital_grafica = capital_df.copy()



    capital_grafica["Acumulado"] = (

        capital_grafica["Capital"]

        .cumsum()

    )



    fig_capital = go.Figure()



    fig_capital.add_trace(

        go.Scatter(

            x=pd.to_datetime(

                capital_grafica["Fecha"]

            ),

            y=capital_grafica["Acumulado"],

            mode="lines+markers",

            name="Capital acumulado"

        )

    )



    fig_capital.update_layout(

        title="Capital aportado en el tiempo",

        height=400

    )



    st.plotly_chart(

        fig_capital,

        use_container_width=True

    )





# =========================
# FIN APP
# =========================

# =========================
# FINANZAS PERSONALES
# =========================

    # =========================
    # FINANZAS PERSONALES
    # =========================

    FINANZAS_COLUMNAS = [

        "Fecha",
        "Tipo",
        "Categoria",
        "Descripcion",
        "Monto"

    ]

    if os.path.exists(FINANZAS_FILE):

        try:

            finanzas_df = pd.read_csv(
                FINANZAS_FILE
            )

            # Verificar si el archivo tiene columnas correctas

            if finanzas_df.empty or not all(
                    col in finanzas_df.columns
                    for col in FINANZAS_COLUMNAS
            ):
                finanzas_df = pd.DataFrame(
                    columns=FINANZAS_COLUMNAS
                )


        except pd.errors.EmptyDataError:

            finanzas_df = pd.DataFrame(
                columns=FINANZAS_COLUMNAS
            )


    else:

        finanzas_df = pd.DataFrame(
            columns=FINANZAS_COLUMNAS
        )
    # =========================
    # REGISTRO INGRESOS/GASTOS
    # =========================

    st.sidebar.subheader(
        "💵 Ingresos y Gastos"
    )

    tipo_movimiento = st.sidebar.selectbox(

        "Tipo",

        [

            "Ingreso",

            "Gasto"

        ]

    )

    if tipo_movimiento == "Ingreso":

        categoria = "Sueldo"



    else:

        categoria = st.sidebar.selectbox(

            "Categoría",

            [

                "🏠 Renta",

                "💳 Tarjeta Crédito",

                "⛽ Gasolina",

                "🚗 Carro",

                "🍔 Comida",

                "📦 Otros",

                "💰 Ahorro"

            ]

        )

    descripcion = st.sidebar.text_input(

        "Descripción"

    )

    monto = st.sidebar.number_input(

        "Monto",

        value=0.0,

        step=50.0

    )

    if st.sidebar.button(
            "Guardar movimiento"
    ):

        if monto > 0:
            nuevo_movimiento = pd.DataFrame(

                [[

                    datetime.now(),

                    tipo_movimiento,

                    categoria,

                    descripcion,

                    monto

                ]],

                columns=finanzas_df.columns

            )

            finanzas_df = pd.concat(

                [

                    finanzas_df,

                    nuevo_movimiento

                ],

                ignore_index=True

            )

            finanzas_df.to_csv(

                FINANZAS_FILE,

                index=False

            )

            st.sidebar.success(

                "Movimiento guardado"

            )

            st.rerun()
# =========================
# RESUMEN FINANZAS
# =========================

# =========================
# RESUMEN FINANZAS MENSUAL
# =========================


st.subheader(
    "💵 Control mensual de dinero"
)



if not finanzas_df.empty:


    # Convertir fechas

    finanzas_df["Fecha"] = pd.to_datetime(
        finanzas_df["Fecha"]
    )


    # =========================
    # SELECTOR MES
    # =========================


    finanzas_df["Mes"] = (
        finanzas_df["Fecha"]
        .dt.strftime("%Y-%m")
    )



    meses_disponibles = sorted(

        finanzas_df["Mes"]
        .unique(),

        reverse=True

    )



    mes_seleccionado = st.selectbox(

        "📅 Seleccionar mes",

        meses_disponibles

    )



    # Filtrar mes seleccionado


    finanzas_mes = finanzas_df[

        finanzas_df["Mes"]

        ==

        mes_seleccionado

    ]




    # =========================
    # CALCULOS DEL MES
    # =========================


    ingresos_mes = finanzas_mes[

        finanzas_mes["Tipo"]

        ==

        "Ingreso"

    ]["Monto"].sum()




    gastos_mes = finanzas_mes[

        finanzas_mes["Tipo"]

        ==

        "Gasto"

    ]["Monto"].sum()




    disponible_mes = (

        ingresos_mes

        -

        gastos_mes

    )




    ahorro_pct = (

        (disponible_mes / ingresos_mes)

        *

        100

        if ingresos_mes > 0

        else 0

    )




    # =========================
    # TARJETAS
    # =========================


    a,b,c,d = st.columns(4)



    a.metric(

        "💰 Ingresos",

        f"${ingresos_mes:,.2f}"

    )



    b.metric(

        "📉 Gastos",

        f"${gastos_mes:,.2f}"

    )



    c.metric(

        "💵 Disponible",

        f"${disponible_mes:,.2f}"

    )



    d.metric(

        "🏦 Capacidad ahorro",

        f"{ahorro_pct:.1f}%"

    )





    # =========================
    # DETALLE DEL MES
    # =========================


    st.subheader(

        f"📜 Movimientos {mes_seleccionado}"

    )



    st.dataframe(

        finanzas_mes.drop(

            columns=["Mes"]

        ),

        use_container_width=True

    )





    # =========================
    # GASTOS POR CATEGORIA
    # =========================


    gastos_categoria = finanzas_mes[

        finanzas_mes["Tipo"]

        ==

        "Gasto"

    ]



    if not gastos_categoria.empty:


        st.subheader(

            "📊 Gastos por categoría"

        )



        resumen_gastos = (

            gastos_categoria

            .groupby(

                "Categoria"

            )["Monto"]

            .sum()

            .reset_index()

        )



        fig = px.pie(

            resumen_gastos,

            values="Monto",

            names="Categoria",

            hole=0.45

        )



        st.plotly_chart(

            fig,

            use_container_width=True

        )




    # =========================
    # EVOLUCION MENSUAL
    # =========================


    st.subheader(

        "📈 Evolución mensual"

    )



    resumen_meses = finanzas_df.copy()



    resumen_meses["Mes"] = (

        resumen_meses["Fecha"]

        .dt.to_period("M")

        .astype(str)

    )



    ingresos = resumen_meses[

        resumen_meses["Tipo"]

        ==

        "Ingreso"

    ].groupby(

        "Mes"

    )["Monto"].sum()



    gastos = resumen_meses[

        resumen_meses["Tipo"]

        ==

        "Gasto"

    ].groupby(

        "Mes"

    )["Monto"].sum()




    evolucion = pd.DataFrame({

        "Ingresos": ingresos,

        "Gastos": gastos

    }).fillna(0)



    evolucion["Disponible"] = (

        evolucion["Ingresos"]

        -

        evolucion["Gastos"]

    )



    st.line_chart(

        evolucion

    )
