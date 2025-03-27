from flask import Flask, render_template, request, redirect, url_for
import sqlite3

app = Flask(__name__)
DATABASE = "database.db"  # Make sure this is the correct database file

# Function to get database connection
def get_db():
    conn = sqlite3.connect(DATABASE)
    conn.row_factory = sqlite3.Row  # Allows column access by name
    return conn

# Route: Home (Display all bookings)
@app.route('/')
def wishlist():
    """
    Fetch all bookings (consultations and installations) and render them on the index page.
    """
    conn = get_db()
    cursor = conn.cursor()
    cursor.execute("SELECT id, username, address, emailaddress FROM bookings")  # Fixed SQL syntax
    bookings = cursor.fetchall()
    conn.close()  # Close the connection
    return render_template('index.html', bookings=bookings)

# Route: Add a new booking
@app.route('/add', methods=['POST'])
def add_booking():
    """
    Add a new booking to the database.
    """
    username = request.form.get('username')
    address = request.form.get('address')
    emailaddress = request.form.get('emailaddress')

    if username and emailaddress and address:
        conn = get_db()
        cursor = conn.cursor()
        cursor.execute(
            "INSERT INTO bookings (emailaddress, address, username) VALUES (?, ?, ?)",
            (emailaddress, address, username),
        )
        conn.commit()
        conn.close()
        return redirect(url_for('wishlist'))  # Fixed the function name

# Route: Delete a booking
@app.route('/delete/<int:booking_id>')
def delete_booking(booking_id):
    """
    Delete a booking from the database.
    """
    conn = get_db()
    cursor = conn.cursor()
    cursor.execute("DELETE FROM bookings WHERE id = ?", (booking_id,))
    conn.commit()
    conn.close()  # Fixed typo
    return redirect(url_for('wishlist'))  # Fixed function name

# Initialize the database
def init_db():
    with sqlite3.connect(DATABASE) as conn:
        cursor = conn.cursor()
        cursor.execute("""
            CREATE TABLE IF NOT EXISTS bookings (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                emailaddress TEXT NOT NULL,
                address TEXT NOT NULL,
                username TEXT NOT NULL
            )
        """)  # Fixed SQL syntax
        conn.commit()

if __name__ == '__main__':
    init_db()  # Ensure database is created before running
    app.run(debug=True)
