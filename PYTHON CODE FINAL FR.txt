#import modules
from tkinter import *
import tkinter as tk
from tkinter import PhotoImage
from PIL import Image, ImageTk  # PIL is used to handle images in Tkinter
from tkinter import ttk
from tkcalendar import Calendar 
import sqlite3
import tkinter.messagebox
from datetime import datetime

# connect to database
conn = sqlite3.connect('appointment_db.db')
#cursor to move around
c = conn.cursor()

ids=[]


class Introduction:
    def __init__(self, master):
        self.master = master
        self.master.title("HIRAYA HOSPITAL APPOINTMENTS")
        self.master.geometry("1200x720+0+0")
        self.master.resizable(False, False)
        
        
        #creating frame for intro
        self.intro_frame = Frame(master, width=1200, height=720, bg='white')
        self.intro_frame.pack()

        self.bg_image = PhotoImage(file="C:\\Users\\christaliza\\Desktop\\hiraya appointment\\images.png")
        self.bg_label = Label(self.intro_frame, image=self.bg_image)
        self.bg_label.place(relwidth=1, relheight=1)
        

        #welcome message
        self.heading = Label(self.intro_frame, text="Welcome to Hiraya Hospital Appointments", font= ('Times 40 bold'), fg='black', bg='gray64')
        self.heading.place(x=100, y=100)
        #hospital introduction
        self.description = Label(self.intro_frame, text="The Hiraya Hospital Appointments System has been designed to help patients book\n"
        "and manage their appointments with ease. Through this system, you can enter your details, \n"
        "select convenient dates and times, and receive immediate confirmation of your visit.\n\n"
        "Let us be your partner in achieving better health. At Hiraya Hospital, \n"
        "your care is in good hands!",
        font=('helvetica 20 normal'), fg='black', bg='gray64')
        self.description.place(x=50, y=260,)

        # Proceed button
        self.proceed_btn = Button(self.intro_frame,text="Proceed to Login", font=('arial 18 bold'), bg='thistle4', fg='white', command=self.go_to_login)
        self.proceed_btn.place(x=450, y=500)

    def go_to_login(self):
        self.intro_frame.destroy()  # Remove introduction frame
        Login(self.master)  # Launch the login

# Create Secretary Table
c.execute("""
CREATE TABLE IF NOT EXISTS tb_secretary (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    username TEXT NOT NULL UNIQUE,
    password TEXT NOT NULL
)
""")

# Create Doctor Table with a Foreign Key for Secretary Assignment
c.execute("""
CREATE TABLE IF NOT EXISTS tb_doctor (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    username TEXT NOT NULL UNIQUE,
    password TEXT NOT NULL,
    specialization TEXT NOT NULL,
    secretary_id INTEGER NOT NULL,
    FOREIGN KEY (secretary_id) REFERENCES tb_secretary(id) ON DELETE CASCADE
)
""")

#create for patients table
c.execute("""
CREATE TABLE IF NOT EXISTS tb_patients (
    appointment_no INT NOT NULL,
    name TEXT NOT NULL,
    age INTEGER NOT NULL,
    gender TEXT NOT NULL,
    location TEXT NOT NULL,
    phone_num TEXT NOT NULL,
    date_cal TEXT NOT NULL,
    time_schedule TEXT NOT NULL
)
""")
conn.commit()

# Create Appointments Table
c.execute("""
CREATE TABLE IF NOT EXISTS tb_appointments (
    appointment_id INTEGER PRIMARY KEY AUTOINCREMENT,
    patient_no INTEGER NOT NULL,
    doctor_name TEXT NOT NULL,
    specialization_role INTEGER,
    appointment_date TEXT NOT NULL,
    appointment_time TEXT NOT NULL,
    FOREIGN KEY (patient_id) REFERENCES tb_patients(appointment_no) ON DELETE CASCADE,
    FOREIGN KEY (doctor_id) REFERENCES tb_users(id) ON DELETE CASCADE
)
""")
conn.commit()

# Initialize secretary table
c.executemany("""
INSERT OR IGNORE INTO tb_secretary (name, username, password) VALUES (?, ?, ?)
""", [
    ("Jhers Marie Torrano", "jhers_secretary", "toji01"),
    ("Maria Mirakel Villavicencio", "mirakel_secretary", "yolo123"),
    ("Anne Marga Canales", "anne_secretary", "anne0107"),
    ("Nadine Joy Lopez", "nadine_secretary", "nads11")
])
conn.commit()

# Fetch secretary IDs for assignments
c.execute("SELECT id FROM tb_secretary ORDER BY id")
secretary_ids = [row[0] for row in c.fetchall()]

# Initialize doctor table with secretary assignments
c.executemany("""
INSERT OR IGNORE INTO tb_doctor (name, username, password, specialization, secretary_id) VALUES (?, ?, ?, ?, ?)
""", [
    ("Dr. Jaspher", "jaspher_cardiology", "jas01", "Cardiology", secretary_ids[0]),
    ("Dr. Ceeje", "ceeje_neurology", "ceeje02", "Neurology", secretary_ids[1]),
    ("Dra. Madeline Marie", "madeline_pediatrics", "mads03", "Pediatrics", secretary_ids[2]),
    ("Dra. Denes M.", "denes_orthopedics", "dens04", "Orthopedics", secretary_ids[3])
])
conn.commit()

# Login Class
class Login:
    def __init__(self, master):
        self.master = master
        self.master.title("Login - Hiraya Hospital Appointment")
        self.master.geometry("700x500+450+150")
        self.master.resizable(False, False)

        # Database connection
        self.conn = sqlite3.connect("appointment_db.db")  # Replace with your database file
        self.cursor = self.conn.cursor()

        self.login_frame = Frame(self.master, bg='cornsilk3', width=700, height=500)
        self.login_frame.pack()
        self.login_image = PhotoImage(file="C:\\Users\\christaliza\\Desktop\\hiraya appointment\\Login.png") 
        self.image_label = Label(master, image=self.login_image, bg="cornsilk3")
        self.image_label.place(x=270, y=135)
    
        # Login Title
        self.heading = Label(self.login_frame, text="ᕼIᖇᗩYᗩ", font='arial 50 bold', bg='cornsilk3')
        self.heading.place(x=230, y=20)
        self.heading = Label(self.login_frame, text="ʜᴏsᴘɪᴛᴀʟ ᴀᴘᴘᴏɪɴᴛᴍᴇɴᴛ", font='arial 20 bold', bg='cornsilk3')
        self.heading.place(x=200, y=90)
        self.heading = Label(self.login_frame, text="Login", font=('Helvetica 24 bold'), bg='cornsilk3')
        self.heading.place(x=285, y=250)

        # Username
        self.username_label = Label(self.login_frame, text="Username:", font=('Helvetica 14'), bg='cornsilk3')
        self.username_label.place(x=100, y=300)
        self.username_entry = Entry(self.login_frame, font=('Helvetica 14'), width=25)
        self.username_entry.place(x=200, y=300)

        # Password
        self.password_label = Label(self.login_frame, text="Password:", font=('Helvetica 14'), bg='cornsilk3')
        self.password_label.place(x=105, y=340)
        self.password_entry = Entry(self.login_frame, font=('Helvetica 14'), width=25, show="*")
        self.password_entry.place(x=200, y=340)

        # Role Selection
        self.role_label = Label(self.login_frame, text="Role:", font=('Helvetica 14'), bg='cornsilk3')
        self.role_label.place(x=150, y=380)
        self.role_combobox = ttk.Combobox(self.login_frame, font=('Helvetica 14'), width=22, state="readonly")
        self.role_combobox['values'] = ("Secretary", "Doctor")
        self.role_combobox.place(x=200, y=380)

        # Specialization (only for Doctors)
        self.specialization=Label(self.login_frame, text="Specialization:", font=('Helvetica', 14), bg='cornsilk3')
        self.specialization.place(x=100, y=420)
        self.specialization_combobox = ttk.Combobox(self.login_frame, font=('Helvetica', 14), state="readonly")
        self.specialization_combobox['values'] = ("Cardiology", "Neurology", "Pediatrics", "Orthopedics")
        self.specialization_combobox.place(x=240, y=420)
        self.specialization_combobox.config(state="disabled")

        # Enable specialization if Doctor is selected
        self.role_combobox.bind("<<ComboboxSelected>>", self.toggle_specialization)


        # Login Button
        self.login_button = Button(self.login_frame, text="Login", font=('Helvetica 14 bold'), bg='lightblue', command=self.validate_login)
        self.login_button.place(x=250, y=460)


        # View Users Button
        self.view_users_button = Button(self.login_frame, text="View Users", font=('Helvetica 14 bold'), bg='lightgray', command=self.view_users)
        self.view_users_button.place(x=350, y=460)
        
    def toggle_specialization(self, event):
        if self.role_combobox.get() == "Doctor":
            self.specialization_combobox.config(state="normal")
        else:
            self.specialization_combobox.set("")
            self.specialization_combobox.config(state="disabled")

    def validate_login(self):
        username = self.username_entry.get()
        password = self.password_entry.get()
        role = self.role_combobox.get()
        specialization = self.specialization_combobox.get()

            # Basic validation for empty fields
        if not username or not password or not role:
            tkinter.messagebox.showerror("Error", "All fields are required!")
            return

        # Additional validation for Doctor role   
        if role == "Doctor" and not specialization:
            tkinter.messagebox.showerror("Error", "Specialization is required for Doctors!")
            return

         # Validate credentials for Secretary
        if role == "Secretary":
            query = """
            SELECT s.id, s.name, d.name AS doctor_name, d.specialization
            FROM tb_secretary s
            LEFT JOIN tb_doctor d ON s.id = d.secretary_id
            WHERE s.username = ? AND s.password = ?
            """
            self.cursor.execute(query, (username, password))
            user_data = self.cursor.fetchone()

            if user_data:
                secretary_id, secretary_name, doctor_name, doctor_specialization = user_data

                if not doctor_name:
                    tkinter.messagebox.showerror("Error", "No doctor assigned to this secretary!")
                    return

                if specialization and specialization != doctor_specialization:
                    tkinter.messagebox.showerror(
                        "Error", 
                        f"Specialization mismatch!\nAssigned Doctor: {doctor_name} ({doctor_specialization})"
                    )
                    return

                tkinter.messagebox.showinfo(
                    "Login Success", 
                    f"Welcome, {secretary_name}!\nAssigned Doctor: {doctor_name} ({doctor_specialization})"
                )
                self.login_frame.destroy()
                SecretaryApp(self.master, doctor_specialization)  # Pass secretary ID to the dashboard
            else:
                tkinter.messagebox.showerror("Error", "Invalid Secretary credentials!")
            return

        # Validate credentials for Doctor
        if role == "Doctor":
            query = """
            SELECT id, name, specialization FROM tb_doctor 
            WHERE username = ? AND password = ? AND specialization = ?
            """
            self.cursor.execute(query, (username, password, specialization))
            user = self.cursor.fetchone()

            if user:
                tkinter.messagebox.showinfo("Login Success", f"Welcome, {user[1]}!")
                self.login_frame.destroy()
                DoctorDashboard(self.master, user[2])  # Pass doctor ID to the dashboard
            else:
                tkinter.messagebox.showerror("Error", "Invalid Doctor credentials or specialization mismatch!")
                
    def view_users(self):
        self.view_users_window = Toplevel(self.master)
        self.view_users_window.title("Doctor-Secretary Assignments")
        self.view_users_window.geometry("900x400")

        # Treeview with an additional hidden column for the user ID
        self.tree = ttk.Treeview(
            self.view_users_window, columns=("UserID", "Doctor", "Specialization", "Secretary"), show="headings"
        )
        self.tree.heading("UserID", text="UserID")  # Hidden column
        self.tree.heading("Doctor", text="Doctor")
        self.tree.heading("Specialization", text="Specialization")
        self.tree.heading("Secretary", text="Secretary Assigned")

        self.tree.column("UserID", width=0, stretch=False)  # Hide the user ID column
        self.tree.pack(fill="both", expand=True)

        # Fetch and display relationships
        query = """
        SELECT d.id AS doctor_id, d.name AS doctor, d.specialization, s.name AS secretary
        FROM tb_doctor d
        JOIN tb_secretary s ON d.secretary_id = s.id
        """
        c.execute(query)
        rows = c.fetchall()
        for row in rows:
            self.tree.insert("", "end", values=row)


        # Back Button
        self.back_button = Button(self.view_users_window, text="Back to Login", font=('Helvetica 12 bold'), bg='lightblue', command=self.back_to_login)
        self.back_button.pack(pady=10)

    def back_to_login(self):
        # Destroy the user management frame
        self.view_users_window.destroy() 
           
# Fetch data for the dashboard
def fetch_data():
    c.execute("SELECT * FROM tb_patients")
    total_patients = c.fetchone()[0]
    return total_patients

class SecretaryApp:
    def __init__(self, master, specialization):
        self.master = master
        self.specialization = specialization  # Logged-in specialization
        self.master.title(f"Secretary Dashboard - {self.specialization}")
        self.master.geometry("1200x720+0+0")
        self.master.resizable(False, False)

        self.left = Frame(master, width=800, height= 720)
        self.left.pack(side=LEFT)

         # Load and display the background image
        self.bg_image = Image.open("C:\\Users\\christaliza\\Desktop\\hiraya appointment\\secretarybg.png")
        self.bg_image = self.bg_image.resize((800, 720), Image.Resampling.LANCZOS)
        self.bg_image_tk = ImageTk.PhotoImage(self.bg_image)

        self.canvas = Canvas(self.left, width=800, height=720)
        self.canvas.pack(fill="both", expand=True)

        # Add the background image to the canvas
        self.canvas.create_image(0, 0, anchor="nw", image=self.bg_image_tk)

        # Add transparent text on top of the image
        self.canvas.create_text( 400, 50, text="ʜɪʀᴀʏᴀ ʜᴏꜱᴘɪᴛᴀʟ ᴀᴘᴘᴏɪɴᴛᴍᴇɴᴛ", font=("gabriela", 40, "bold"), fill="pink3" )

        
        self.right = Frame(master, width=400, height=720, bg='linen')
        self.right.pack(side=RIGHT)

        #labels for the window
        #appointment log 
        self.logs_label = Label(self.right, text="Appointment Logs", font=('arial 26 bold'), bg='linen')
        self.logs_label.place(x=20, y=15)
        self.logs_box = tk.Text(self.right, width=45, height=30)
        self.logs_box.place(x=20, y=85)
        #Labels and entry
        #appointment number
        self.appointment_no = Label(self.left, text="Appointment No. :", font=('arial 14 bold'), fg='black')
        self.appointment_no.place(x=30, y=110)
        AppointmentSlots = [
            "1", "2", "3", "4", "5", "6", "7", "8", "9", "10",
            "11", "12", "13", "14", "15"
        ]
        self.appnum = tk.StringVar(value=self.appointment_no)
        self.appnum.set(AppointmentSlots[0])
        self.appnum_menu = tk.OptionMenu(self.left, self.appnum, *AppointmentSlots)
        self.appnum_menu.config(width=10, font='arial 11 bold')
        self.appnum_menu.place(x=225, y=110)

        #patients name
        self.name = Label(self.left, text="Patient's Name:", font=('arial 14 bold'), fg='black')
        self.name.place(x=30, y=150)
        self.name_ent = Entry(self.left, width=20, font="arial 14")
        self.name_ent.place(x=200, y=150)
        #age
        self.agelbl = Label(self.left, text="Patient's Age:", font=('arial 14 bold'), fg='black')
        self.agelbl.place(x=455, y=150)
        self.age_ent = Entry(self.left, width=10, font="arial 14")
        self.age_ent.place(x=595, y=150)
        #genderlist
        GenderList = ["Select Gender", "Male", "Female"]
        #option menu
        self.ugenderent = tk.StringVar()
        self.ugenderent.set(GenderList[0])

        self.opt = tk.OptionMenu(self.left, self.ugenderent, *GenderList)
        self.opt.config(width=15, font='arial 11 bold')
        self.opt.place (x=525, y=190)

        #callback method
        def callback (*args):
            for i in range(len(GenderList)):
                if GenderList[i] == self.ugenderent.get():
                    self.gender_ent = GenderList[i]
                    break
        self.ugenderent.trace("w", callback)
        #location
        self.locationlbl = Label(self.left, text="Location:", font=('arial 14 bold'), fg='black')
        self.locationlbl.place(x=30, y=190)
        self.location_ent = Entry(self.left, width=20, font="arial 14")
        self.location_ent.place(x=150, y=190)
        #phone number
        self.phone = Label(self.left, text="Phone Number:", font=('arial 14 bold'), fg='black')
        self.phone.place(x=30, y=240)
        self.phone_ent = Entry(self.left, width=20, font="arial 14")
        self.phone_ent.place(x=200, y=240)
        #appointment date time
        self.date = Label(self.left, text="Appointment Date:", font=('arial 14 bold'), fg='black')
        self.date.place(x=30, y=280)
        self.date_calent = Calendar(self.left, selectmode='day', date_pattern='yyyy-mm-dd')
        self.date_calent.place(x=280, y=280)
        self.utime = Label(self.master, text="Appointment Time:", font=('arial 14 bold'), fg='black')
        self.utime.place(x=30, y=490)
        # Time Entry
        TimeSlots = ["Select Time",
            "09:00 AM", "09:30 AM", "10:00 AM", "10:30 AM",
            "11:00 AM", "11:30 AM", "12:00 PM", "12:30 PM",
            "01:00 PM", "01:30 PM", "02:00 PM", "02:30 PM",
            "03:00 PM", "03:30 PM", "04:00 PM"
        ]
        self.time_var = tk.StringVar(value=self.utime)
        self.time_var.set(TimeSlots[0])
        self.time_menu = tk.OptionMenu(self.left, self.time_var, *TimeSlots)
        self.time_menu.config(width=10, font='arial 11 bold')
        self.time_menu.place(x=235, y=490)

        # Specialization selection
        self.specialization_label = Label(self.left, text="Specialization:", font=('arial 14 bold'), fg='black')
        self.specialization_label.place(x=30, y=540)
        self.specialization_combobox = ttk.Combobox(self.left, font=('Arial', 12), state="readonly")
        self.specialization_combobox['values'] = ("Cardiology", "Neurology", "Pediatrics", "Orthopedics")
        self.specialization_combobox.place(x=190, y=540)

        # Doctor selection based on specialization
        self.doctor_label = Label(self.left, text="Doctor Assigned:", font=('arial 14 bold'), fg='black')
        self.doctor_label.place(x=30, y=590)
        self.doctor_combobox = ttk.Combobox(self.left, font=('Arial', 12), state="readonly")
        self.doctor_combobox.place(x=210, y=590)
        self.specialization_combobox.bind("<<ComboboxSelected>>", self.update_doctor_list)


        #buttons
        self.addbtn = Button(self.left, text="ADD APPOINTMENT", bg='gray', fg='white', font=("Arial", 12, "bold"), command=self.addApp)
        self.addbtn.place(x=350, y=630)
        
        self.displaybtn = Button(self.left, text="DISPLAY APPOINTMENTS", bg='gray', fg='white', font=("Arial", 12, "bold"), command=self.displayAppointments)
        self.displaybtn.place(x=550, y=630)

         # Exit Button
        self.exitbtn = Button(self.right, text="EXIT",  font=("Arial", 12, "bold"), command=self.master.destroy)
        self.exitbtn.place(x=200, y=610)

        # Logout Button
        self.logout_button = tk.Button(self.right, text="LOG OUT", font=('Helvetica', 12, "bold"), command=self.logout)
        self.logout_button.place (x=290, y=610)

    def update_doctor_list(self, event=None):
        """Update doctor combobox based on selected specialization."""
        SPECIALIZATION_TO_DOCTOR = {
            "Cardiology": "Dr. Jaspher",
            "Neurology": "Dr. Ceeje",
            "Pediatrics": "Dra. Madeline Marie",
            "Orthopedics": "Dra. Denes M.",
        }
        selected_specialization = self.specialization_combobox.get()
        assigned_doctor = SPECIALIZATION_TO_DOCTOR.get(selected_specialization, "None")
        
        # Update doctor_combobox values and set default
        self.doctor_combobox['values'] = (assigned_doctor,)
        self.doctor_combobox.set(assigned_doctor if assigned_doctor != "None" else "")

    def logout(self):
        """Logout and return to the login screen"""
        self.left.destroy()
        self.right.destroy()
        Login(self.master)


    def addApp(self):
        self.val1 = self.appnum.get()
        self.val2 = self.name_ent.get()
        self.val3 = self.age_ent.get()
        self.val4 = self.ugenderent.get()
        self.val5 = self.location_ent.get()
        self.val6 = self.phone_ent.get()
        self.val7 = self.date_calent.get_date()
        self.val8 = self.time_var.get()
        self.val9 = self.specialization_combobox.get()
        self.val10 = self.doctor_combobox.get()

        # Check if any input field is empty
        if self.val1 == '' or self.val2 == '' or self.val3 == '' or self.val4 == "Select Gender" or self.val5 == '' or self.val6 == '' or self.val7 == '' or self.val8 == "Select Time" or self.val9 == '' or self.val10 =='':
            tkinter.messagebox.showinfo("Warning", "Please fill up all the details")
            return

        # Validate phone number & age
        if not self.val6.isdigit() or len(self.val6) != 11:
            tkinter.messagebox.showerror("Invalid Phone Number", "Phone number must be numeric and exactly 11 digits.")
            return

        if not self.val3.isdigit() or int(self.val3) <= 0:
            tkinter.messagebox.showerror("Error", "Age must be a positive integer.")
            return
        if not self.val10 or self.val10 == "None":
            tkinter.messagebox.showerror("Invalid Doctor", "Please select a valid specialization and doctor.")
            return

        try:
            # Database operations
            with sqlite3.connect('appointment_db.db') as conn:
                cursor = conn.cursor()

                
                # Check for conflicts with date and time
                cursor.execute("""
                    SELECT * FROM tb_patients WHERE appointment_no = ? AND date_cal = ? AND time_schedule = ?
                """, (self.val1, self.val7, self.val8))
                if cursor.fetchone():
                    tkinter.messagebox.showerror(
                        "Conflict Detected",
                        f"An appointment with number {self.val1} is already scheduled on {self.val7} at {self.val8}."
                    )
                    return

                # Insert appointment under the logged-in specialization
                sql_patient = """
                    INSERT INTO tb_patients (appointment_no, name, age, gender, location, phone_num, date_cal, time_schedule) 
                    VALUES (?, ?, ?, ?, ?, ?, ?, ?)
                """
                cursor.execute(sql_patient, (self.val1, self.val2, self.val3, self.val4, self.val5, self.val6, self.val7, self.val8))

                sql_appointment = """
                    INSERT INTO tb_appointments (patient_no, doctor_name, specialization_role, appointment_date, appointment_time)
                    VALUES (?, ?, ?, ?, ?)
                """
                cursor.execute(sql_appointment, (self.val1, self.val10, self.val9, self.val7, self.val8))

                conn.commit()
                tkinter.messagebox.showinfo("Success", f"Appointment for {self.val2} added under {self.specialization}.")
                self.logs_box.insert(tk.END, f"Appointment: {self.val1} | Patient: {self.val2} | Date: {self.val7} | Time: {self.val8} | Doctor: {self.val10}\n")
                self.clearFields()

        except Exception as e:
            tkinter.messagebox.showerror("Database Error", str(e))

    def clearFields(self):
        # Clear all input fields
        self.appnum.set("1")
        self.name_ent.delete(0, END)
        self.age_ent.delete(0, END)
        self.ugenderent.set("Select Gender")
        self.location_ent.delete(0, END)
        self.phone_ent.delete(0, END)
        self.time_var.set("Select Time")
        self.specialization_combobox.set(" ")
        self.doctor_combobox.set(" ")

    # Function to fetch all appointments with doctor names from the database
    def fetch_appointments_by_specialization(self):
        """Fetch and display appointments based on the secretary's assigned specialization."""
        try:
            connection = sqlite3.connect('appointment_db.db')
            cursor = connection.cursor()
            cursor.execute("""
                SELECT p.appointment_no, p.name, p.age, p.gender, p.location, p.phone_num, 
                    p.date_cal, p.time_schedule, a.doctor_name
                FROM tb_patients p
                JOIN tb_appointments a ON p.appointment_no = a.patient_no
                JOIN tb_doctor d ON a.doctor_name = d.name
                WHERE d.specialization = ?
            """, (self.specialization,))
            rows = cursor.fetchall()
            connection.close()

            # Populate the Treeview with fetched data
            for row in self.tree.get_children():
                self.tree.delete(row)

            for appointment in rows:
                self.tree.insert('', 'end', values=appointment)
        except sqlite3.Error as e:
            tkinter.messagebox.showerror("Database Error", f"An error occurred: {e}")


    # Function to display appointments
    def displayAppointments(self):
        """Display appointments filtered by specialization."""
        # New window for displaying appointments
        self.display_window = Toplevel(self.master)
        self.display_window.title(f"Appointments - {self.specialization}")
        self.display_window.geometry("1000x600")

        # Search Section
        search_frame = Frame(self.display_window)
        search_frame.pack(fill=X, padx=10, pady=10)

        search_label = Label(search_frame, text="Search Patient Name: ", font= "arial 12")
        search_label.pack(side=LEFT, padx=5)

        self.search_entry = Entry(search_frame, width=30)
        self.search_entry.pack(side=LEFT, padx=5)

        search_button = Button(search_frame, text="Search", bg='lightyellow', command=self.searchApp)
        search_button.pack(side=LEFT, padx=5)

        reset_button = Button(search_frame, text="Reset", bg='lightblue', command=self.resetSearch)
        reset_button.pack(side=LEFT, padx=5)

        # Treeview for displaying appointments
        self.tree = ttk.Treeview(self.display_window, columns=(
            "appointment_no", "name", "age", "gender", "location", "phone", "date", "time"), show="headings")
        self.tree.heading("appointment_no", text="Appointment No")
        self.tree.heading("name", text="Name")
        self.tree.heading("age", text="Age")
        self.tree.heading("gender", text="Gender")
        self.tree.heading("location", text="Location")
        self.tree.heading("phone", text="Phone")
        self.tree.heading("date", text="Date")
        self.tree.heading("time", text="Time")

        self.tree.column("appointment_no", width=60)
        self.tree.column("name", width=100)
        self.tree.column("age", width=50)
        self.tree.column("gender", width=70)
        self.tree.column("location", width=100)
        self.tree.column("phone", width=100)
        self.tree.column("date", width=100)
        self.tree.column("time", width=100)

        self.tree.pack(fill=BOTH, expand=True)

        # Insert fetched data into the tree (filtered by specialization)
        self.fetch_appointments_by_specialization()
        
        self.specialization_combobox.set(self.specialization)  # Reset to logged-in specialization

        # Buttons for managing appointments inside the displayAppointments
        self.add_btn = Button(self.display_window, text="Add", bg='lightblue', command=self.display_window.destroy)
        self.add_btn.pack(side=LEFT, padx=10, pady=10)

        self.update_btn = Button(self.display_window, text="Update", bg='lightgreen', command=self.updateApp)
        self.update_btn.pack(side=LEFT, padx=10, pady=10)

        self.cancel_btn = Button(self.display_window, text="Cancel", bg='lightpink', command=self.cancelApp)
        self.cancel_btn.pack(side=LEFT, padx=10, pady=10)

        self.done_btn = Button(self.display_window, text="Done", bg='lightcoral', command=self.doneApp)
        self.done_btn.pack(side=LEFT, padx=10, pady=10)

        self.exit_btn = Button(self.display_window, text="Exit", bg='gray', fg='white', command=self.display_window.destroy)
        self.exit_btn.pack(side=LEFT, padx=10, pady=10)

        # Button to open appointments window
        self.appointments_btn = Button(self.display_window, text="View Appointments", bg='mediumpurple1',  command=self.openAppointmentsWindow)
        self.appointments_btn.pack(side=LEFT, padx=10, pady=10)

    def openAppointmentsWindow(self):
        # Destroy the previous window (if open)
        if hasattr(self, 'display_window') and self.display_window.winfo_exists():
            self.display_window.destroy()
        # Create a new top-level window for appointments
        self.appointments_window = Toplevel(self.master)
        self.appointments_window.title("Appointments")
        self.appointments_window.geometry("800x600")

        # Create a treeview to display appointments
        self.appointments_tree = ttk.Treeview(self.appointments_window, columns=("patient_no", "doctor_name", "specialization_role", "appointment_date", "appointment_time"), show="headings")
        
        self.appointments_tree.heading("patient_no", text="Patient No")
        self.appointments_tree.heading("doctor_name", text="Doctor Name")
        self.appointments_tree.heading("specialization_role", text="Specialization")
        self.appointments_tree.heading("appointment_date", text="Date")
        self.appointments_tree.heading("appointment_time", text="Time")

        self.appointments_tree.column("patient_no", width=100)
        self.appointments_tree.column("doctor_name", width=150)
        self.appointments_tree.column("specialization_role", width=150)
        self.appointments_tree.column("appointment_date", width=100)
        self.appointments_tree.column("appointment_time", width=100)

        self.appointments_tree.pack(fill=BOTH, expand=True)

        # Fetch and display appointment data from tb_appointments table
        self.displayAppointmentsData()

        # Add buttons for delete and close
        buttons_frame = Frame(self.appointments_window)
        buttons_frame.pack(pady=10)

        delete_btn = Button(buttons_frame, text="Delete Appointment", command=self.deleteSelectedAppointment)
        delete_btn.pack(side=LEFT, padx=5)

        close_btn = Button(buttons_frame, text="Close", command=self.appointments_window.destroy)
        close_btn.pack(side=LEFT, padx=5)

    def displayAppointmentsData(self):
        # Fetch appointment data from the database
        connection = sqlite3.connect('appointment_db.db')
        cursor = connection.cursor()
        cursor.execute("SELECT patient_no, doctor_name, specialization_role, appointment_date, appointment_time FROM tb_appointments")
        rows = cursor.fetchall()
        connection.close()

        
        # Insert data into the treeview
        for row in rows:
            self.appointments_tree.insert('', 'end', values=row)

    def deleteSelectedAppointment(self):
        # Get the selected item
        selected_item = self.appointments_tree.selection()
        if not selected_item:
            tkinter.messagebox.showwarning("No Selection", "Please select an appointment to delete.")
            return

        # Confirm deletion
        confirm = tkinter.messagebox.askyesno("Confirm Delete", "Are you sure you want to delete this appointment?")
        if not confirm:
            return

        # Retrieve patient_no from the selected item
        patient_no = self.appointments_tree.item(selected_item)['values'][0]

        try:
            # Delete from database
            connection = sqlite3.connect('appointment_db.db')
            cursor = connection.cursor()
            cursor.execute("DELETE FROM tb_appointments WHERE patient_no = ?", (patient_no,))
            cursor.execute("DELETE FROM tb_patients WHERE appointment_no = ?", (patient_no,))
            connection.commit()
            connection.close()

            # Remove the item from the treeview
            self.appointments_tree.delete(selected_item)

            tkinter.messagebox.showinfo("Success", f"Appointment for patient {patient_no} deleted successfully.")

            # Destroy the window after deleting the appointment
            self.appointments_window.destroy()
        except Exception as e:
            tkinter.messagebox.showerror("Error", f"Could not delete the appointment. Error: {str(e)}")

   # Function to update an appointment
    def updateApp(self):
        # Check if the Treeview widget exists
        if not self.tree or not self.tree.winfo_exists():
            tkinter.messagebox.showerror("Error", "Appointment list is not available.")
            return
        # Get the selected appointment from the treeview
        selected_item = self.tree.focus()
        if not selected_item:
            tkinter.messagebox.showwarning("Warning", "Please select an appointment to update.")
            return
        
        appointment_data = self.tree.item(selected_item, "values")
        if not appointment_data:
            tkinter.messagebox.showerror("Error", "Unable to retrieve appointment details.")
            return
        
        # Close the display window
        if self.display_window and self.display_window.winfo_exists():
            self.display_window.withdraw()
        
        # Store the original appointment number for reference
        self.original_appnum = appointment_data[0]
        # Populate the main dashboard fields with the selected appointment's data
        self.appnum.set(appointment_data[0])
        self.name_ent.delete(0, END)
        self.name_ent.insert(0, appointment_data[1])
        self.age_ent.delete(0, END)
        self.age_ent.insert(0, appointment_data[2])
        self.ugenderent.set(appointment_data[3])
        self.location_ent.delete(0, END)
        self.location_ent.insert(0, appointment_data[4])
        self.phone_ent.delete(0, END)
        self.phone_ent.insert(0, appointment_data[5])
        self.date_calent.get_date()
        self.time_var.set(appointment_data[7])
        
        # Change the ADD button to SAVE button
        self.addbtn.configure(text="SAVE CHANGES", command=self.saveChanges)

    def saveChanges(self):
        updated_data = (
            self.appnum.get(),
            self.name_ent.get(),
            self.age_ent.get(),
            self.ugenderent.get(),
            self.location_ent.get(),
            self.phone_ent.get(),
            self.date_calent.get_date(),
            self.time_var.get()
        )

        # Validate inputs
        if '' in updated_data or updated_data[3] == "Select Gender" or updated_data[7] == "Select Time":
            tkinter.messagebox.showwarning("Warning", "Please fill up all the details correctly.")
            return
        if not updated_data[5].isdigit() or len(updated_data[5]) != 11:
            tkinter.messagebox.showerror("Invalid Phone Number", "Phone number must be numeric and exactly 11 digits.")
            return
        if not updated_data[2].isdigit() or int(updated_data[2]) <= 0:
            tkinter.messagebox.showerror("Error", "Age must be a positive integer.")
            return

        try:
        # Check for conflicting appointments
            with sqlite3.connect('appointment_db.db') as conn:
                cursor = conn.cursor()
                conflict_query = """
                    SELECT * FROM tb_patients 
                    WHERE date_cal = ? AND time_schedule = ? AND appointment_no != ?
                """
                conflicts = cursor.execute(conflict_query, (updated_data[6], updated_data[7], self.original_appnum)).fetchall()

            if conflicts:
                tkinter.messagebox.showerror(
                    "Conflict",
                    f"The selected time slot ({updated_data[7]}) on {updated_data[6]} is already booked. "
                    "Please choose another time or date."
                )
                return

            # Proceed with the update
            with sqlite3.connect('appointment_db.db') as conn:
                cursor = conn.cursor()
                query = """
                    UPDATE tb_patients SET 
                    appointment_no = ?, name = ?, age = ?, gender = ?, location = ?, 
                    phone_num = ?, date_cal = ?, time_schedule = ? 
                    WHERE appointment_no = ?
                """
                cursor.execute(query, (
                    updated_data[0],  # New appointment number
                    updated_data[1],  # Name
                    updated_data[2],  # Age
                    updated_data[3],  # Gender
                    updated_data[4],  # Location
                    updated_data[5],  # Phone
                    updated_data[6],  # Date
                    updated_data[7],  # Time
                    self.original_appnum  # Original appointment number for WHERE condition
                ))
                conn.commit()

                # Update Treeview
                selected_item = self.tree.selection()
                self.tree.item(selected_item, values=updated_data)

                # Log and Notify
                timestamp = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
                tkinter.messagebox.showinfo("Success", f"Appointment for {updated_data[1]} has been updated.")
                self.logs_box.insert(
                    tk.END,
                    f"\n[{timestamp}] Updated appointment for {updated_data[1]} on {updated_data[6]} at {updated_data[7]}"
                )

            # Reopen the display window
            if self.display_window and not self.display_window.winfo_exists():
                self.displayAppointments()

        
            
            # Reset SAVE button back to ADD APPOINTMENT
            self.addbtn.configure(text="ADD APPOINTMENT", command=self.addApp)

            # Clear input fields
            self.clearFields()
            # Refresh the display window Treeview
            self.refreshDisplayAppointments()

        except Exception as e:
            tkinter.messagebox.showerror("Database Error", str(e))

    def refreshDisplayAppointments(self):
        try:
            # Clear the Treeview
            for item in self.tree.get_children():
                self.tree.delete(item)

            # Load data from the database
            with sqlite3.connect('appointment_db.db') as conn:
                cursor = conn.cursor()
                cursor.execute("SELECT * FROM tb_patients")
                rows = cursor.fetchall()

            # Insert updated data into the Treeview
            for row in rows:
                self.tree.insert("", tk.END, values=row)

        except Exception as e:
            tkinter.messagebox.showerror("Database Error", f"Unable to refresh appointments: {str(e)}")

    def clearFields(self):
        # Clear all input fields
        self.appnum.set("1")
        self.name_ent.delete(0, END)
        self.age_ent.delete(0, END)
        self.ugenderent.set("Select Gender")
        self.location_ent.delete(0, END)
        self.phone_ent.delete(0, END)
        self.time_var.set("Select Time")

    # Function to mark done the appointment
    def doneApp(self):
        # Get the selected item in the tree view
        selected_item = self.tree.focus()
        if not selected_item:
            tkinter.messagebox.showerror("Error", "No appointment selected!")
            return

        # Retrieve the values of the selected appointment
        appointment = self.tree.item(selected_item, 'values')

        # Confirm deletion with the user
        confirm = tkinter.messagebox.askyesno(
            "Confirm",
            f"Are you sure you want to delete the appointment of {appointment[1]}?"
        )
        if not confirm:
            return

        try:
            # Connect to the database and delete the appointment
            with sqlite3.connect('appointment_db.db') as connection:
                cursor = connection.cursor()
                cursor.execute("DELETE FROM tb_patients WHERE appointment_no=? AND date_cal=? AND time_schedule=?", 
                            (appointment[0], appointment[6], appointment[7]))
                if cursor.rowcount == 0:  # Check if any row was deleted
                    tkinter.messagebox.showerror("Error", "Unable to delete the appointment. It may not exist.")
                    return
                connection.commit()

            # Remove the appointment from the tree view
            self.tree.delete(selected_item)
            tkinter.messagebox.showinfo("Success", f"Appointment Number {appointment[0]} done!")

        except sqlite3.Error as e:
            tkinter.messagebox.showerror("Database Error", f"An error occurred while deleting the appointment: {e}")

    # Function to cancel an appointment
    def cancelApp(self):
        # Get the selected item in the tree view
        selected_item = self.tree.focus()
        if not selected_item:
            tkinter.messagebox.showerror("Error", "No appointment selected!")
            return

        # Retrieve the values of the selected appointment
        appointment = self.tree.item(selected_item, 'values')

        try:
            # Connect to the database and delete the appointment
            with sqlite3.connect('appointment_db.db') as connection:
                cursor = connection.cursor()
                cursor.execute("DELETE FROM tb_patients WHERE appointment_no=? AND date_cal=? AND time_schedule=?", 
                            (appointment[0], appointment[6], appointment[7]))
                if cursor.rowcount == 0:  # Check if any row was deleted
                    tkinter.messagebox.showerror("Error", "Unable to delete the appointment. It may not exist.")
                    return
                connection.commit()

            # Remove the appointment from the tree view
            self.tree.delete(selected_item)
            tkinter.messagebox.showinfo("Success", f"Appointment Number {appointment[0]} canceled!")

        except sqlite3.Error as e:
            tkinter.messagebox.showerror("Database Error", f"An error occurred while deleting the appointment: {e}")


    # Function to search appointments
    def searchApp(self):
        search_term = self.search_entry.get().strip()

        if not search_term:
            tkinter.messagebox.showwarning("Warning", "Please enter a name to search.")
            return

        try:
            # Connect to database and perform search
            connection = sqlite3.connect('appointment_db.db')
            cursor = connection.cursor()
            cursor.execute("SELECT * FROM tb_patients WHERE name LIKE ?", ('%' + search_term + '%',))
            results = cursor.fetchall()
            connection.close()

            # Clear treeview and display search results
            for item in self.tree.get_children():
                self.tree.delete(item)

            if results:
                for result in results:
                    self.tree.insert('', 'end', values=result)
            else:
                tkinter.messagebox.showinfo("No Results", "No matching appointments found.")

        except sqlite3.Error as e:
            tkinter.messagebox.showerror("Database Error", f"An error occurred while searching: {e}")

# Function to reset the search and reload all appointments
    def resetSearch(self):
        self.search_entry.delete(0, END)
        # Clear treeview
        for item in self.tree.get_children():
            self.tree.delete(item)

        # Reload all appointments
        appointments = self.fetch_appointments()
        for appointment in appointments:
            self.tree.insert('', 'end', values=appointment) 


class DoctorDashboard:
    def __init__(self, master, specialization):
        self.master = master
        self.specialization = specialization
        self.master.title("Doctor Dashboard - Hiraya Hospital Appointment")
        self.master.geometry("1000x600+200+100")
        self.master.resizable(False, False)

        self.dashboard_frame = tk.Frame(self.master, bg='wheat2')
        self.dashboard_frame.pack(fill=tk.BOTH, expand=True)

        self.heading = tk.Label(self.dashboard_frame, text="Doctor Dashboard", font='consolas 32 bold', bg='wheat2')
        self.heading.pack(pady=20)

        # Search section
        self.search_label = tk.Label(self.dashboard_frame, text="Search Patient:", font=('Helvetica', 14), bg='wheat2')
        self.search_label.pack(padx=20)
        self.search_entry = tk.Entry(self.dashboard_frame, font=('Helvetica', 14), width=30)
        self.search_entry.pack(pady=10)

        # Search and Reset buttons
        button_frame = tk.Frame(self.dashboard_frame, bg='wheat2')
        button_frame.pack(pady=10)

        self.search_button = tk.Button(button_frame, text="Search", font=('Helvetica', 14), command=self.search_appointments)
        self.search_button.pack(side=tk.LEFT, padx=5)

        self.reset_button = tk.Button(button_frame, text="Reset", font=('Helvetica', 14), command=self.load_appointments)
        self.reset_button.pack(side=tk.LEFT, padx=5)

        # Treeview to display patient appointments
        self.tree = ttk.Treeview(self.dashboard_frame, columns=("appointment_no", "name", "age", "gender", "location", "phone", "date", "time"), show="headings")
        self.tree.heading("appointment_no", text="Appointment No")
        self.tree.heading("name", text="Name")
        self.tree.heading("age", text="Age")
        self.tree.heading("gender", text="Gender")
        self.tree.heading("location", text="Location")
        self.tree.heading("phone", text="Phone")
        self.tree.heading("date", text="Date")
        self.tree.heading("time", text="Time")

        self.tree.column("appointment_no", width=100)
        self.tree.column("name", width=150)
        self.tree.column("age", width=50)
        self.tree.column("gender", width=70)
        self.tree.column("location", width=150)
        self.tree.column("phone", width=100)
        self.tree.column("date", width=100)
        self.tree.column("time", width=100)

        self.tree.pack(fill=tk.BOTH, expand=True)

        # "Mark as Done" button
        self.done_button = tk.Button(self.dashboard_frame, text="Mark as Done", font=('Helvetica', 14), command=self.mark_done)
        self.done_button.pack(pady=10)

        # Logout Button
        self.logout_button = tk.Button(self.dashboard_frame, text="Logout", font=('Helvetica', 14), command=self.logout)
        self.logout_button.pack(pady=5)

        # Exit Button
        self.exit_button = tk.Button(self.dashboard_frame, text="Exit", font=('Helvetica', 12), bg="indianred4", fg="black", command=self.exit_dashboard)
        self.exit_button.pack(pady=5)

        self.load_appointments()  # Load all appointments initially

    def load_appointments(self):
        """Fetch and display appointments specific to the doctor's specialization."""
        try:
            connection = sqlite3.connect('appointment_db.db')
            cursor = connection.cursor()

            # Query to fetch appointments based on the doctor's specialization (using doctor name instead of doctor_id)
            query = """
                SELECT a.patient_no, p.name, p.age, p.gender, p.location, p.phone_num, p.date_cal, p.time_schedule
                FROM tb_appointments a
                INNER JOIN tb_patients p ON a.patient_no = p.appointment_no
                INNER JOIN tb_doctor d ON a.doctor_name = d.name  -- Join using doctor_name
                WHERE d.specialization = ?
            """
            cursor.execute(query, (self.specialization,))
            appointments = cursor.fetchall()
            connection.close()
        except sqlite3.Error as e:
            tkinter.messagebox.showerror("Error", f"Database error: {e}")
            return

        # Clear previous data in the tree view
        for row in self.tree.get_children():
            self.tree.delete(row)

        # Populate the tree with the fetched data
        for appointment in appointments:
            self.tree.insert('', 'end', values=appointment)


    def search_appointments(self):
        """Search for appointments based on patient name and the doctor's specialization."""
        name = self.search_entry.get().strip()  # Get the input from the search entry
        if not name:
            tkinter.messagebox.showerror("Error", "Please enter a patient's name to search.")
            return

        try:
            connection = sqlite3.connect('appointment_db.db')
            cursor = connection.cursor()

            # Query to search appointments by patient name and doctor's specialization
            query = """
                SELECT a.patient_no, p.name, p.age, p.gender, p.location, p.phone_num, p.date_cal, p.time_schedule
                FROM tb_appointments a
                INNER JOIN tb_patients p ON a.patient_no = p.appointment_no
                INNER JOIN tb_doctor d ON a.doctor_name = d.name
                WHERE p.name LIKE ? AND d.specialization = ?
            """
            cursor.execute(query, ('%' + name + '%', self.specialization))
            results = cursor.fetchall()
            connection.close()

            # Handle no results found
            if not results:
                tkinter.messagebox.showerror("Error", "No matching appointments found.")
                return

            # Clear previous data in the tree view
            for row in self.tree.get_children():
                self.tree.delete(row)

            # Insert search results into the tree
            for result in results:
                self.tree.insert('', 'end', values=result)

        except sqlite3.Error as e:
            tkinter.messagebox.showerror("Database Error", f"An error occurred: {e}")


    def mark_done(self):
        """Mark the selected appointment as done and remove it from the dashboard."""
        selected_item = self.tree.focus()  # Get the selected Treeview item
        if not selected_item:
            tkinter.messagebox.showerror("Error", "Please select an appointment to mark as done.")
            return

        appointment = self.tree.item(selected_item, "values")  # Get the data of the selected appointment
        appointment_no = appointment[0]  # Assuming `patient_no` is the first column in the Treeview data

        try:
            connection = sqlite3.connect('appointment_db.db')
            cursor = connection.cursor()

            # Remove the appointment from the database
            cursor.execute("DELETE FROM tb_appointments WHERE patient_no = ?", (appointment_no,))
            cursor.execute("DELETE FROM tb_patients WHERE appointment_no = ?", (appointment_no,))
            connection.commit()

            # Remove the selected item from the Treeview
            self.tree.delete(selected_item)

            # Show success message
            tkinter.messagebox.showinfo("Success", f"Appointment {appointment_no} marked as done.")
        except sqlite3.Error as e:
            tkinter.messagebox.showerror("Database Error", f"An error occurred: {e}")
        finally:
            connection.close()

    def exit_dashboard(self):
        """Exit the Doctor Dashboard application."""
        response = tkinter.messagebox.askyesno("Exit", "Are you sure you want to exit?")
        if response:  # If user confirms exit
            self.master.destroy()  # Closes the main application window


    def logout(self):
        """Logout and return to the login screen"""
        self.dashboard_frame.destroy()
        Login(self.master)

# Main Application
root = tk.Tk()
login = Introduction(root)
root.mainloop()
