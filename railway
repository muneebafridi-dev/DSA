# ================= IMPORT =================
import streamlit as st
import sqlite3
import pandas as pd

# ================= DATABASE =================
conn = sqlite3.connect("railway.db", check_same_thread=False)
c = conn.cursor()

# ================= CREATE TABLES =================
c.execute("""
CREATE TABLE IF NOT EXISTS users (
    username TEXT,
    password TEXT,
    role TEXT
)
""")

c.execute("""
CREATE TABLE IF NOT EXISTS trains (
    train_number TEXT,
    train_name TEXT,
    date TEXT,
    time TEXT,
    start TEXT,
    end TEXT
)
""")

# 🔹 UPDATED tickets table (class & price added)
c.execute("""
CREATE TABLE IF NOT EXISTS tickets (
    train_number TEXT,
    username TEXT,
    passenger_name TEXT,
    gender TEXT,
    class TEXT,
    price INTEGER
)
""")

conn.commit()

# ================= DEFAULT ADMIN =================
c.execute("INSERT OR IGNORE INTO users VALUES (?,?,?)", ("admin", "Afridi123", "admin"))
conn.commit()

# ================= SESSION =================
if "login" not in st.session_state:
    st.session_state.login = False
    st.session_state.role = ""
if "username" not in st.session_state:
    st.session_state.username = ""

# ================= SIGNUP =================
st.sidebar.subheader("Signup (User Only)")
new_user = st.sidebar.text_input("New Username")
new_pass = st.sidebar.text_input("New Password", type="password")

if st.sidebar.button("Signup"):
    if new_user and new_pass:
        c.execute("SELECT * FROM users WHERE username=?", (new_user,))
        if c.fetchone():
            st.sidebar.error("Username already exists")
        else:
            c.execute("INSERT INTO users VALUES (?,?,?)", (new_user, new_pass, "user"))
            conn.commit()
            st.sidebar.success("Signup successful! Now login.")
    else:
        st.sidebar.error("Enter both username and password")

# ================= LOGIN =================
st.title("CECOS Reservation System")
username_input = st.text_input("Username")
password_input = st.text_input("Password", type="password")
role_option = st.selectbox("Login As", ["Admin", "User"])

if not st.session_state.login:
    if st.button("Login"):
        c.execute(
            "SELECT role FROM users WHERE username=? AND password=?",
            (username_input, password_input)
        )
        data = c.fetchone()

        if data and data[0].lower() == role_option.lower():
            st.session_state.login = True
            st.session_state.role = data[0].lower()
            st.session_state.username = username_input
            st.success(f"Login Successful as {role_option}")
        else:
            st.error("Wrong credentials or role")

# ================= MENU =================
if st.session_state.login:

    if st.sidebar.button("Logout"):
        st.session_state.login = False
        st.session_state.role = ""
        st.session_state.username = ""

    # ================= ADMIN =================
    if st.session_state.role == "admin":
        menu = st.sidebar.selectbox(
            "Menu",
            ["Add Train", "View Trains", "View Tickets", "Cancel Ticket", "Delete Train"]
        )

        if menu == "Add Train":
            tn = st.text_input("Train Number")
            name = st.text_input("Train Name")
            date = st.text_input("Date")
            time = st.text_input("Time")
            start = st.text_input("From")
            end = st.text_input("To")

            if st.button("Add Train"):
                c.execute(
                    "INSERT INTO trains VALUES (?,?,?,?,?,?)",
                    (tn, name, date, time, start, end)
                )
                conn.commit()
                st.success("Train Added")

        elif menu == "View Trains":
            st.dataframe(pd.read_sql("SELECT * FROM trains", conn))

        elif menu == "View Tickets":
            st.dataframe(pd.read_sql("SELECT * FROM tickets", conn))

        elif menu == "Cancel Ticket":
            tn = st.text_input("Train Number")
            passenger = st.text_input("Passenger Name")
            if st.button("Cancel Ticket"):
                c.execute(
                    "DELETE FROM tickets WHERE train_number=? AND passenger_name=?",
                    (tn, passenger)
                )
                conn.commit()
                st.success("Ticket Cancelled")

        elif menu == "Delete Train":
            tn = st.text_input("Train Number")
            if st.button("Delete Train"):
                c.execute("DELETE FROM trains WHERE train_number=?", (tn,))
                conn.commit()
                st.success("Train Deleted")

    # ================= USER =================
    else:
        menu = st.sidebar.selectbox(
            "Menu",
            ["View Trains", "Book Ticket", "View My Tickets", "Cancel Ticket"]
        )

        if menu == "View Trains":
            st.dataframe(pd.read_sql("SELECT * FROM trains", conn))

        elif menu == "Book Ticket":
            tn = st.text_input("Train Number")
            passenger_name = st.text_input("Passenger Name")
            gender = st.radio("Gender", ["Male", "Female"])
            travel_class = st.selectbox("Class", ["Economy", "Business"])

            # 🔹 Seat limits
            ECONOMY_LIMIT = 30
            BUSINESS_LIMIT = 20

            # Count already booked seats
            c.execute(
                "SELECT COUNT(*) FROM tickets WHERE train_number=? AND class=?",
                (tn, travel_class)
            )
            booked_seats = c.fetchone()[0]

            # 🔹 Price logic
            price = 1000 if travel_class == "Economy" else 2000
            st.info(f"Ticket Price: Rs. {price}")
            st.info(f"Seats booked in {travel_class}: {booked_seats} / {'30' if travel_class=='Economy' else '20'}")

            if st.button("Book Ticket"):
                if travel_class == "Economy" and booked_seats >= ECONOMY_LIMIT:
                    st.error("❌ Economy class is full")
                elif travel_class == "Business" and booked_seats >= BUSINESS_LIMIT:
                    st.error("❌ Business class is full")
                elif tn and passenger_name and gender:
                    username = st.session_state.username
                    c.execute(
                        "INSERT INTO tickets VALUES (?,?,?,?,?,?)",
                        (tn, username, passenger_name, gender, travel_class, price)
                    )
                    conn.commit()
                    st.success("✅ Ticket Booked Successfully!")
                else:
                    st.error("Enter all details")

        elif menu == "View My Tickets":
            username = st.session_state.username
            df = pd.read_sql(
                "SELECT * FROM tickets WHERE username=?",
                conn,
                params=(username,)
            )
            if df.empty:
                st.info("No tickets booked yet.")
            else:
                st.dataframe(df)

        elif menu == "Cancel Ticket":
            tn = st.text_input("Train Number")
            passenger_name = st.text_input("Passenger Name")
            if st.button("Cancel Ticket"):
                username = st.session_state.username
                c.execute(
                    "DELETE FROM tickets WHERE train_number=? AND username=? AND passenger_name=?",
                    (tn, username, passenger_name)
                )
                conn.commit()
                st.success("Ticket Cancelled")
