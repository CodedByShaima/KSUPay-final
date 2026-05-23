import tkinter as tk
import tkinter.messagebox
import sqlite3
import random as rand
import datetime
import csv
from tkinter.messagebox import askokcancel
import datetime



def create_database():
    conn = sqlite3.connect('ksupay.db')
    cursor = conn.cursor()

    cursor.execute('''
    CREATE TABLE IF NOT EXISTS Users (
        id TEXT PRIMARY KEY,
        first_name TEXT,
        last_name TEXT,
        password TEXT,
        email TEXT,
        phone TEXT,
        role TEXT
    )''')

    cursor.execute('''
    CREATE TABLE IF NOT EXISTS Wallets (
        wallet_number TEXT PRIMARY KEY,
        owner_id TEXT,
        wallet_type TEXT,
        balance REAL,
        created_at TEXT,
        FOREIGN KEY (owner_id) REFERENCES Users (id)
    )''')

    cursor.execute("DELETE FROM Wallets")
    cursor.execute("DELETE FROM Users")

    users_to_add = [
        ('4411223344', 'Ahmed', 'Ali', '123456', 'ahmed@student.ksu.edu.sa', '0501111111', 'Student'),
        ('4422334455', 'Sara', 'Khalid', 'sara123', 'sara@student.ksu.edu.sa', '0522222222', 'Student'),
        ('4433445566', 'Omar', 'Nasser', 'omar123', 'omar@student.ksu.edu.sa', '0533333333', 'Student'),
        ('4444556677', 'Fatima', 'Yousef', 'fati123', 'fatima@student.ksu.edu.sa', '0544444444', 'Student'),
        ('1010101010', 'Admin', 'System', 'admin99', 'admin@ksupay.com', '0500000000', 'Admin'),
    ]

    for user in users_to_add:
        cursor.execute('''
            INSERT OR IGNORE INTO Users (id, first_name, last_name, password, email, phone, role)
            VALUES (?, ?, ?, ?, ?, ?, ?)
        ''', user)

    wallets_to_add = [
        ('1111111111', '4411223344', 'Student', 2500, '2025-01-01 10:00:00'),
        ('2222222222', '4422334455', 'Student', 500, '2025-01-02 11:00:00'),
        ('3333333333', '4433445566', 'Student', 3200, '2025-01-03 12:00:00'),
        ('4444444444', '4444556677', 'Student', 100, '2025-01-04 13:00:00'),
        ('5555555555', 'bookstore', 'KSU', 5000, '2025-01-01 09:00:00'),
        ('6666666666', 'housing', 'KSU', 10000, '2025-01-01 09:30:00'),
        ('7777777777', 'cafeteria', 'KSU', 2500, '2025-01-01 10:00:00'),
        ('8888888888', 'library', 'KSU', 0, '2025-01-01 11:00:00'),
    ]

    for wallet in wallets_to_add:
        cursor.execute('''
            INSERT OR IGNORE INTO Wallets (wallet_number, owner_id, wallet_type, balance, created_at)
            VALUES (?, ?, ?, ?, ?)
        ''', wallet)

    conn.commit()
    conn.close()

class SignUp:
    def __init__(self):
        self.root = tk.Tk()
        self.root.title("○ Sign Up to KSU-pay ○")
        self.root.geometry("500x600")
        self.root.configure(background="light gray")

        tk.Label(self.root, text="○ ○ Sign Up to KSU-pay ○ ○", font=("Times 20 bold", 17), bg="light gray").pack(pady=(5, 5))

        tk.Label(self.root, text="First Name", bg="light gray").pack(pady=(15, 15))
        self.Fname = tk.Entry(self.root)
        self.Fname.pack()

        tk.Label(self.root, text="Last Name", bg="light gray").pack(pady=(15, 15))
        self.Lname = tk.Entry(self.root)
        self.Lname.pack()

        tk.Label(self.root, text="Student ID (exactly 10 digits)", bg="light gray").pack(pady=(15, 15))
        self.ID = tk.Entry(self.root)
        self.ID.pack()

        tk.Label(self.root, text="Password (at least 6 digits)", bg="light gray").pack(pady=(15, 15))
        self.password = tk.Entry(self.root)
        self.password.pack()

        tk.Label(self.root, text="Email (xxxxxx@student.ksu.edu.sa)", bg="light gray").pack(pady=(15, 15))
        self.email = tk.Entry(self.root)
        self.email.pack()

        tk.Label(self.root, text="Phone Number (05xxxxxxxx)", bg="light gray").pack(pady=(15, 15))
        self.phone_num = tk.Entry(self.root)
        self.phone_num.pack()

        tk.Button(self.root, text="Submit", command=self.signUp_submit).pack(pady=(15, 15))
        tk.Button(self.root, text="Login", command=self.go_to_login).pack(pady=(15, 15))

        self.root.mainloop()

    def signUp_submit(self):
        first_name = self.Fname.get()
        last_name = self.Lname.get()
        stu_id = self.ID.get()
        pas = self.password.get()
        em = self.email.get()
        ph = self.phone_num.get()

        if first_name == "":
            tk.messagebox.showinfo("error", "first name is required")
            return
        if last_name == "":
            tk.messagebox.showinfo("error", "last name is required")
            return
        if stu_id == "" or len(stu_id) != 10 or not stu_id.isdigit():
            tk.messagebox.showinfo("error", "Student ID is required and must be 10 digits")
            return
        if pas == "" or len(pas) < 6:
            tk.messagebox.showinfo("error", "Password is required and must be at least 6 digits")
            return
        if em == "" or not em.endswith("@student.ksu.edu.sa"):
            tk.messagebox.showinfo("error", "Email is required in the form xxxxxx@student.ksu.edu.sa")
            return
        if ph == "" or not ph.startswith("05") or not ph.isdigit() or len(ph) != 10:
            tk.messagebox.showinfo("error", "Phone number is required: must start with 05 and be 10 digits")
            return

        try:
            conn = sqlite3.connect("ksupay.db")
            cursor = conn.cursor()
            cursor.execute("SELECT * FROM Users WHERE id = ?", (stu_id,))
            if cursor.fetchone():
                askokcancel(title="error", message="Student exists\nHave an account? please log in")
                self.clear_fields()
                conn.close()
                return

            wallet_num = rand.randint(10 ** 9, (10 ** 10) - 1)
            balance = 1000
            date_time = datetime.datetime.now()

            cursor.execute(
                f"Insert Into Users values('{stu_id}','{first_name}','{last_name}','{pas}','{em}','{ph}','Student')")

            cursor.execute(
                f"Insert Into Wallets values('{wallet_num}','{stu_id}','Student','{balance}','{date_time}')")

            conn.commit()
            conn.close()
            tk.messagebox.showinfo("success", "Student and wallet added successfully")
            self.clear_fields()

        except sqlite3.IntegrityError:
            tk.messagebox.showinfo("error", "Unexpected error in the database")
        except Exception as e:
            tk.messagebox.showinfo("error", f"Unexpected error: {e}")

    def clear_fields(self):
        self.Fname.delete(0, tk.END)
        self.Lname.delete(0, tk.END)
        self.ID.delete(0, tk.END)
        self.password.delete(0, tk.END)
        self.email.delete(0, tk.END)
        self.phone_num.delete(0, tk.END)

    def go_to_login(self):
        self.root.destroy()
        LogIn()


class LogIn:
    def __init__(self):
        self.window = tk.Tk()
        self.window.title("KSUPay - Login")
        self.window.geometry("400x400")

        tk.Label(self.window, text="KSUPay Login", font=("Arial", 16, "bold")).pack(pady=20)

        tk.Label(self.window, text='Enter Student or Admin ID').pack()
        self.ent_id = tk.Entry(self.window)
        self.ent_id.pack(pady=5)

        tk.Label(self.window, text='Enter Password').pack()
        self.ent_pass = tk.Entry(self.window, show="*")  # عشان يخفي الباسوورد
        self.ent_pass.pack(pady=5)

        tk.Button(self.window, text="Login", width=15, command=self.validate_login).pack(pady=20)

        # عشان يرجع للsignUP
        tk.Button(self.window, text="Back to Sign Up", width=15, command=self.go_back_to_signup).pack(pady=5)

        self.window.mainloop()

    def validate_login(self):
        user_id = self.ent_id.get()
        password = self.ent_pass.get()

        if not user_id.isdigit() or len(user_id) != 10:
            tk.messagebox.showwarning("Input Error", "ID must be exactly 10 digits.")
            return

        try:
            conn = sqlite3.connect('ksupay.db')
            cursor = conn.cursor()
            cursor.execute("SELECT role, first_name FROM Users WHERE id=? AND password=?", (user_id, password))
            user_data = cursor.fetchone()
            conn.close()

            if user_data:
                role = user_data[0]
                name = user_data[1]
                tk.messagebox.showinfo("Success", f"Welcome {name}!")
                self.window.destroy()

                if role == 'Student':
                    self.open_student_wallet(user_id)
                else:
                    self.open_admin_window()
            else:
                tk.messagebox.showerror("Error", "Invalid ID or Password")

        except sqlite3.Error as e:
            tk.messagebox.showerror("DB Error", f"Error: {e}")

    def open_student_wallet(self, student_id):
        StudentWallet(student_id)

    def open_admin_window(self):
        Admin()

    def go_back_to_signup(self):
        self.window.destroy()
        SignUp()


class Admin:
    def __init__(self):
        self.window = tk.Tk()
        self.window.title("KSUPay - Admin")
        self.window.geometry("400x400")

        # to access DB
        try:
            self.conn = sqlite3.connect('ksupayy.db')
            self.cursor = self.conn.cursor()
        except sqlite3.Error as e:
            self.connecting_error = tk.messagebox.showerror("DB Error", f"Error: {e}")

        # frames
        self.total_frame = tk.Frame(self.window)
        self.total_frame.pack(pady=10)

        self.create_frame = tk.Frame(self.window)
        self.create_frame.pack(pady=10)

        self.stipend_frame = tk.Frame(self.window)
        self.stipend_frame.pack(pady=10)

        self.cash_frame = tk.Frame(self.window)
        self.cash_frame.pack(pady=10)

        self.backup_frame = tk.Frame(self.window)
        self.backup_frame.pack(pady=10)

        self.back_frame = tk.Frame(self.window)
        self.back_frame.pack(pady=10)

        # label of page
        self.label1 = tk.Label(self.window, text="KSUPay Admin", font=("Arial", 16, "bold")).pack(pady=20)

        # views the total balance for KSY
        KsuBalance = self.view_total_balance()
        self.balance = tk.Label(self.total_frame, text=f"Total KSU Balance: {KsuBalance} SR").pack()

        # Entity name field
        self.prompt=tk.Label(self.create_frame,text=" Enter Student ID or KSU Entity Name (e.g. Bookstore, Cafeteria) \n to creat wallet for",).pack()
        self.Entityname_or_ID = tk.Entry(self.create_frame)
        self.Entityname_or_ID.pack(pady=5)

        # BUTTONS
        # takes Entity name than call (creat wallet) method
        self.creat_wallte = tk.Button(self.create_frame, text="Creat wallet", command=self.CreatWallet_Entity).pack(
            pady=5)

        # Pay Stipends button
        self.stipens = tk.Button(self.stipend_frame, text="Pay Stipends", command=self.pay_stipends).pack(pady=5)

        # Cash Out button
        self.cash_out = tk.Button(self.cash_frame, text="Cash Out", command=self.cash_out).pack(pady=5)

        # backup button
        self.backup = tk.Button(self.backup_frame, text="Back up", command=self.backup).pack(pady=5)

        # Back button
        self.back = tk.Button(self.back_frame, text="Back", command=self.goBack).pack(pady=5)

        self.window.mainloop()

    def goBack(self):
        self.window.destroy()
        SignUp()  # goes to log in window

    def view_total_balance(self):
        try:
            self.cursor.execute("SELECT SUM(balance) FROM Wallets WHERE wallet_type = ?", ("KSU",))
            total = self.cursor.fetchone()[0]
            return total
        except sqlite3.Error as e:
            tk.messagebox.showerror("DB Error", f"Error: {e}")

    def CreatWallet_Entity(self):
        try:
            value = self.Entityname_or_ID.get().strip()
            # checks if field is empty or not
            if not value:
                tk.messagebox.showerror("Error", "Please enter entity name!")
                self.Entityname_or_ID.delete(0, tk.END)  # to delete the entry for rentring
                return

            # detrimense wether the entry is a student or entity and acts accordingly
            if value.isdigit():
                self.cursor.execute("SELECT id FROM Users WHERE id = ?", (value,))  # checkes if student exist in db
                if not self.cursor.fetchone():
                    tk.messagebox.showerror("Error", "Student ID not found!!")
                    self.Entityname_or_ID.delete(0, tk.END)  # to delete the entry for rentring
                    return
                wallet_type = "Student"
            else:
                value = value.lower()

                # checkes if entity exist in db
                self.cursor.execute("SELECT id FROM Users WHERE id = ? AND role = 'KSU'", (value,))
                if self.cursor.fetchone():
                    tk.messagebox.showerror("Error", "Entity already Eixist!!")
                    self.Entityname_or_ID.delete(0, tk.END)  # to delete the entry for rentring
                    return
                wallet_type = "KSU"

            # when adding wallet for student this statment checkes if student already have wallet or not if not then add wallet
            self.cursor.execute("SELECT wallet_number FROM Wallets WHERE owner_id = ?", (value,))
            if self.cursor.fetchone():
                tk.messagebox.showerror("Error", f"The student with ID: {value} already has a wallet!")
                return

            # to insert entity name into user tabel before creating a wallet for it
            if wallet_type == "KSU":
                self.cursor.execute('''
                INSERT OR IGNORE INTO Users (id, first_name, last_name, password, email, phone, role)
                VALUES (?, ?, ?, ?, ?, ?, ?)
            ''', (value, value.capitalize(), 'KSU Entity', '', '', '', 'KSU'))

            # to genrate a 10-digit wallet number
            self.wallet_number = str(rand.randint(1000000000, 9999999999))

            # to record release date
            today = datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")

            # insert new wallet into Wallet tabel in db
            self.cursor.execute("""INSERT INTO Wallets (wallet_number, owner_id, wallet_type, balance, created_at)
                VALUES (?, ?, ?, ?, ?)""", (self.wallet_number, value, wallet_type, 0, today))
            self.conn.commit()
            tk.messagebox.showinfo("Success", f"Wallet has been created successfully!")

        except sqlite3.Error as e:
            tk.messagebox.showerror("DB Error", f"Error: {e}")

    def pay_stipends(self):
        try:
            self.cursor.execute("UPDATE Wallets SET balance = balance + 1000 WHERE wallet_type = ?", ("Student",))
            self.conn.commit()
            tk.messagebox.showinfo("Success", "Stipends paid to all students!")
        except sqlite3.Error as e:
            tk.messagebox.showerror("DB Error", f"Error: {e}")

    def cash_out(self):
        try:
            self.cursor.execute("UPDATE Wallets SET balance = 0 WHERE wallet_type = ?", ("KSU",))
            self.conn.commit()
            tk.messagebox.showinfo("Success", "Cash Out completed!")
        except sqlite3.Error as e:
            tk.messagebox.showerror("DB Error", f"Error: {e}")

    def backup(self):
        user_f = None
        wallet_f = None
        try:
            # opening two files
            user_f = open('backup_users.csv', 'w', newline='')
            wallet_f = open('backup_wallets.csv', 'w', newline='')

            # writting into the files
            user_write = csv.writer(user_f)
            wallet_write = csv.writer(wallet_f)
            # backing up users
            user_write.writerow(['id', 'Fname', 'Lname', 'password', 'email', 'phone', 'role'])
            self.cursor.execute("SELECT * FROM Users")
            for row in self.cursor.fetchall():
                user_write.writerow(row)

            # backing up wallets
            wallet_write.writerow(['wallet_number', 'owner_id', 'wallet_type', 'balance', 'created_at'])
            self.cursor.execute("SELECT * FROM Wallets")
            for row in self.cursor.fetchall():
                wallet_write.writerow(row)
            # success messaage
            tk.messagebox.showinfo("Success", "Backup completed!")
        except Exception as e:
            tk.messagebox.showerror("DB Error", f"Error: {e}")
        finally:
            # closign files after writting
            if user_f:
                user_f.close()
            if wallet_f:
                wallet_f.close()

class StudentWallet:
    def __init__(self, student_id):
        self.student_id = student_id
        self.window = tk.Tk()
        self.window.title("KSUPay - Student Wallet")
        self.window.geometry("450x450")
        self.window.configure(background="light gray")

        # Fetch wallet info from DB
        conn = sqlite3.connect('ksupay.db')
        cursor = conn.cursor()
        cursor.execute("SELECT wallet_number, balance FROM Wallets WHERE owner_id = ?", (self.student_id,))
        wallet = cursor.fetchone()
        conn.close()

        self.wallet_number = wallet[0]
        self.current_balance = wallet[1]

        # --- Wallet Info Display ---
        tk.Label(self.window, text="○ Student Wallet ○", font=("Times", 17, "bold"),
                 background="light gray").pack(pady=(15, 5))

        info_frame = tk.Frame(self.window, background="light gray")
        info_frame.pack(pady=10)

        tk.Label(info_frame, text="Wallet Number:", font=("Arial", 11, "bold"),
                 background="light gray").grid(row=0, column=0, sticky="w", padx=10)
        tk.Label(info_frame, text=str(self.wallet_number), font=("Arial", 11),
                 background="light gray").grid(row=0, column=1, sticky="w")

        tk.Label(info_frame, text="Current Balance:", font=("Arial", 11, "bold"),
                 background="light gray").grid(row=1, column=0, sticky="w", padx=10, pady=5)
        self.balance_label = tk.Label(info_frame, text=f"{self.current_balance} SR",
                                      font=("Arial", 11), background="light gray")
        self.balance_label.grid(row=1, column=1, sticky="w")

        # --- Transfer Section ---
        tk.Label(self.window, text="Target Wallet Number:", background="light gray").pack(pady=(15, 3))
        self.target_wallet_entry = tk.Entry(self.window, width=25)
        self.target_wallet_entry.pack()

        tk.Label(self.window, text="Amount to Pay (SR):", background="light gray").pack(pady=(10, 3))
        self.amount_entry = tk.Entry(self.window, width=25)
        self.amount_entry.pack()

        # --- Buttons ---
        tk.Button(self.window, text="Pay", width=15, command=self.pay).pack(pady=(20, 5))
        tk.Button(self.window, text="Back", width=15, command=self.go_back).pack(pady=5)

        self.window.mainloop()

    def pay(self):
        target = self.target_wallet_entry.get().strip()
        amount_str = self.amount_entry.get().strip()

        # --- Input Validation ---
        if not target or not target.isdigit():
            tk.messagebox.showwarning("Input Error", "Please enter a valid wallet number (digits only).")
            return

        if not amount_str:
            tk.messagebox.showwarning("Input Error", "Please enter an amount.")
            return

        try:
            amount = float(amount_str)
            if amount <= 0:
                tk.messagebox.showwarning("Input Error", "Amount must be greater than zero.")
                return
        except ValueError:
            tk.messagebox.showwarning("Input Error", "Amount must be a number.")
            return

        # --- Database Operations ---
        try:
            conn = sqlite3.connect('ksupay.db')
            cursor = conn.cursor()

            # Check target wallet exists
            cursor.execute("SELECT balance FROM Wallets WHERE wallet_number = ?", (target,))
            target_wallet = cursor.fetchone()

            if not target_wallet:
                tk.messagebox.showerror("Error", "Wallet number does not exist.")
                conn.close()
                return

            # Check student can't pay themselves
            if str(target) == str(self.wallet_number):
                tk.messagebox.showwarning("Error", "You cannot transfer to your own wallet.")
                conn.close()
                return

            # Check sufficient balance
            cursor.execute("SELECT balance FROM Wallets WHERE wallet_number = ?", (str(self.wallet_number),))
            current = cursor.fetchone()[0]

            if amount > current:
                tk.messagebox.showerror("Error", "There is not enough money")
                conn.close()
                return

            # Perform transfer
            cursor.execute("UPDATE Wallets SET balance = balance - ? WHERE wallet_number = ?",
                           (amount, str(self.wallet_number)))
            cursor.execute("UPDATE Wallets SET balance = balance + ? WHERE wallet_number = ?",
                           (amount, target))
            conn.commit()

            # Refresh displayed balance
            cursor.execute("SELECT balance FROM Wallets WHERE wallet_number = ?", (str(self.wallet_number),))
            self.current_balance = cursor.fetchone()[0]
            self.balance_label.config(text=f"{self.current_balance} SR")
            conn.close()

            self.target_wallet_entry.delete(0, tk.END)
            self.amount_entry.delete(0, tk.END)
            tk.messagebox.showinfo("Success", f"{amount} SR transferred successfully!")

        except sqlite3.Error as e:
            tk.messagebox.showerror("DB Error", f"Database error: {e}")

    def go_back(self):
        self.window.destroy()
        SignUp()
if __name__ == "__main__":
    create_database()
    SignUp()