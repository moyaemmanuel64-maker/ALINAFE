import streamlit as st
import pandas as pd
import plotly.express as px
from datetime import date

# ==============================
# APP CONFIG
# ==============================
st.set_page_config(page_title="PHARMAMOYA", layout="wide")

st.title("💊 PHARMAMOYA PHARMACY SYSTEM")

# ==============================
# SESSION STATE (NO LOGIN / NO TOML / NO SECURITY)
# ==============================
if "stock" not in st.session_state:
    st.session_state.stock = pd.DataFrame(columns=["Medicine", "Batch", "Expiry", "Qty"])

if "sales" not in st.session_state:
    st.session_state.sales = pd.DataFrame(columns=["Medicine", "Qty", "Price", "Total", "Date"])

# ==============================
# MENU
# ==============================
menu = st.sidebar.selectbox(
    "Navigation",
    ["Dashboard", "Stock Management", "Sales", "Expiry Alerts"]
)

# ==============================
# DASHBOARD
# ==============================
if menu == "Dashboard":

    st.header("📊 Dashboard")

    total_stock = len(st.session_state.stock)
    total_sales = st.session_state.sales["Total"].sum() if not st.session_state.sales.empty else 0

    col1, col2 = st.columns(2)

    col1.metric("📦 Stock Items", total_stock)
    col2.metric("💰 Total Sales", f"MK {total_sales}")

# ==============================
# STOCK MANAGEMENT
# ==============================
elif menu == "Stock Management":

    st.header("📦 Stock Management")

    with st.form("stock_form"):
        med = st.text_input("Medicine")
        batch = st.text_input("Batch Number")
        expiry = st.date_input("Expiry Date")
        qty = st.number_input("Quantity", min_value=1)

        submit = st.form_submit_button("Add Stock")

        if submit:
            new_stock = {
                "Medicine": med,
                "Batch": batch,
                "Expiry": expiry,
                "Qty": qty
            }

            st.session_state.stock = pd.concat(
                [st.session_state.stock, pd.DataFrame([new_stock])],
                ignore_index=True
            )

            st.success("Stock added successfully!")

    st.subheader("Stock Records")
    st.dataframe(st.session_state.stock, use_container_width=True)

# ==============================
# SALES
# ==============================
elif menu == "Sales":

    st.header("🧾 Sales System")

    with st.form("sales_form"):
        med = st.text_input("Medicine")
        qty = st.number_input("Quantity", min_value=1)
        price = st.number_input("Price", min_value=0.0)

        total = qty * price

        submit = st.form_submit_button("Record Sale")

        if submit:
            new_sale = {
                "Medicine": med,
                "Qty": qty,
                "Price": price,
                "Total": total,
                "Date": date.today()
            }

            st.session_state.sales = pd.concat(
                [st.session_state.sales, pd.DataFrame([new_sale])],
                ignore_index=True
            )

            st.success("Sale recorded!")

    st.subheader("Sales Records")
    st.dataframe(st.session_state.sales, use_container_width=True)

# ==============================
# EXPIRY ALERTS
# ==============================
elif menu == "Expiry Alerts":

    st.header("⚠️ Expiry Alerts")

    if st.session_state.stock.empty:
        st.info("No stock available")
    else:
        stock = st.session_state.stock.copy()
        stock["Expiry"] = pd.to_datetime(stock["Expiry"])

        today = pd.to_datetime(date.today())

        expired = stock[stock["Expiry"] < today]
        expiring_soon = stock[
            (stock["Expiry"] >= today) &
            (stock["Expiry"] <= today + pd.Timedelta(days=30))
        ]

        st.subheader("❌ Expired Medicines")
        st.dataframe(expired)

        st.subheader("⏳ Expiring Soon")
        st.dataframe(expiring_soon)
