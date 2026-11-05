import tkinter
from tkinter import messagebox
from datetime import datetime, date


ROOMS = []
for floor in range(1, 6):
    for number in range(1, 8):
        ROOMS.append("COE2" + str(floor) + "0" + str(number))

PROGRAMS = ["Manufacturing Engineering", "Mechanical Engineering",
            "Mechatronics Engineering", "Computer Engineering",
            "Civil Engineering", "Industrial Engineering",
            "Electronics Engineering", "Electrical Engineering"]

YEAR_LEVELS = ["1st Year", "2nd Year", "3rd Year", "4th Year"]
SECTIONS = ["A", "B", "C", "D"]

OPEN_HOUR = 7   
CLOSE_HOUR = 20  

ADMIN_USERNAME = "admin"
ADMIN_PASSWORD = "admin123"

BG = "#d4e4f7"       
BG_PANEL = "#e8e8e8"  


def hour_text(hour):
    """Turn a 24-hour number (like 13) into text (like 1:00 PM)."""
    if hour == 12:
        return "12:00 PM"
    if hour > 12:
        return str(hour - 12) + ":00 PM"
    return str(hour) + ":00 AM"



START_TIMES = []
END_TIMES = []
HOUR_NUMBER = {}
for h in range(OPEN_HOUR, CLOSE_HOUR + 1):
    HOUR_NUMBER[hour_text(h)] = h
    if h < CLOSE_HOUR:
        START_TIMES.append(hour_text(h))
    if h > OPEN_HOUR:
        END_TIMES.append(hour_text(h))


def get_floor(room):
    """COE2301 -> floor 3 (the 5th character)."""
    return int(room[4])


def only_letters_and_spaces(text):
    for ch in text:
        if not (ch.isalpha() or ch == " " or ch == "." or ch == "-"):
            return False
    return True


def valid_id_number(text):
    """Digits and dashes only, at least 5 digits."""
    digits = 0
    for ch in text:
        if ch.isdigit():
            digits += 1
        elif ch != "-":
            return False
    return digits >= 5


class ReservationManager:
    def __init__(self):
        self.reservations = []   
        self.next_id = 1

    def is_available(self, room, day, start, end):
        """True if no reservation overlaps the given time."""
        for r in self.reservations:
            if r["room"] == room and r["date"] == day:
                if start < r["end"] and end > r["start"]:
                    return False
        return True

    def add_reservation(self, info):
        info["id"] = self.next_id
        self.next_id += 1
        self.reservations.append(info)

    def cancel_reservation(self, res_id):
        for r in self.reservations:
            if r["id"] == res_id:
                self.reservations.remove(r)
                return True
        return False

    def reservations_for_room(self, room, day):
        found = []
        for r in self.reservations:
            if r["room"] == room and r["date"] == day:
                found.append(r)
        return found


manager = ReservationManager()


def reservation_line(r):
    return (str(r["id"]) + " | " + r["date"] + " | " + r["room"] + " | " +
            hour_text(r["start"]) + "-" + hour_text(r["end"]) + " | " +
            r["name"] + " (" + r["role"] + ")")



class MainWindow:
    def __init__(self, window):
        self.window = window
        window.title("Engineering Laboratory Reservation System")
        window.geometry("860x640")
        window.configure(bg=BG)

        tkinter.Label(window, text="College of Engineering",
                      font=("Arial", 10), bg=BG).pack(pady=(8, 0))
        tkinter.Label(window, text="Laboratory Reservation System",
                      font=("Arial", 16, "bold"), bg=BG).pack(pady=(0, 6))

        body = tkinter.Frame(window, bg=BG)
        body.pack(fill="both", expand=True, padx=10)

        left = tkinter.Frame(body, bg=BG_PANEL, bd=2, relief="groove")
        left.pack(side="left", fill="both", expand=True, padx=(0, 5))
        right = tkinter.Frame(body, bg=BG_PANEL, bd=2, relief="groove")
        right.pack(side="left", fill="both", expand=True, padx=(5, 0))

        self.build_form(left)
        self.build_availability(right)

        bottom = tkinter.Frame(window, bg=BG)
        bottom.pack(fill="x", padx=10, pady=8)
        tkinter.Button(bottom, text="Reserve", width=12,
                       command=self.reserve).pack(side="left", padx=3)
        tkinter.Button(bottom, text="Clear", width=12,
                       command=self.clear_form).pack(side="left", padx=3)
        tkinter.Button(bottom, text="Administrator Login", width=18,
                       command=self.open_admin_login).pack(side="right", padx=3)

        self.show_role_fields()
        self.refresh_rooms()

   
    def make_row(self, parent, text):
        row = tkinter.Frame(parent, bg=BG_PANEL)
        row.pack(fill="x", padx=8, pady=2)
        label = tkinter.Label(row, text=text, width=18, anchor="w", bg=BG_PANEL)
        label.pack(side="left")
        return row, label

    def build_form(self, parent):
        tkinter.Label(parent, text="Reservation Form", font=("Arial", 11, "bold"),
                      bg=BG_PANEL).pack(pady=4)

       
        self.role = tkinter.StringVar(value="Student")
        row, label = self.make_row(parent, "I am a:")
        tkinter.Radiobutton(row, text="Student", variable=self.role, value="Student",
                            bg=BG_PANEL, command=self.show_role_fields).pack(side="left")
        tkinter.Radiobutton(row, text="Teacher", variable=self.role, value="Teacher",
                            bg=BG_PANEL, command=self.show_role_fields).pack(side="left")

        
        self.person_frame = tkinter.Frame(parent, bg=BG_PANEL)
        self.person_frame.pack(fill="x")

        row, label = self.make_row(self.person_frame, "Full name:")
        self.name_entry = tkinter.Entry(row, width=28)
        self.name_entry.pack(side="left")

        row, self.id_label = self.make_row(self.person_frame, "Student number:")
        self.id_entry = tkinter.Entry(row, width=28)
        self.id_entry.pack(side="left")

        row, self.program_label = self.make_row(self.person_frame, "Program:")
        self.program = tkinter.StringVar(value="Select program")
        tkinter.OptionMenu(row, self.program, *PROGRAMS).pack(side="left")

        
        self.student_frame = tkinter.Frame(self.person_frame, bg=BG_PANEL)
        self.year = tkinter.StringVar(value=YEAR_LEVELS[0])
        row, label = self.make_row(self.student_frame, "Year level:")
        tkinter.OptionMenu(row, self.year, *YEAR_LEVELS).pack(side="left")
        self.section = tkinter.StringVar(value=SECTIONS[0])
        row, label = self.make_row(self.student_frame, "Section:")
        tkinter.OptionMenu(row, self.section, *SECTIONS).pack(side="left")

        
        tkinter.Label(parent, text="Reservation Details", font=("Arial", 11, "bold"),
                      bg=BG_PANEL).pack(pady=(10, 2))

        row, label = self.make_row(parent, "Date (YYYY-MM-DD):")
        self.date_entry = tkinter.Entry(row, width=14)
        self.date_entry.insert(0, date.today().strftime("%Y-%m-%d"))
        self.date_entry.pack(side="left")

        row, label = self.make_row(parent, "Laboratory room:")
        self.room = tkinter.StringVar(value=ROOMS[0])
        tkinter.OptionMenu(row, self.room, *ROOMS).pack(side="left")

        row, label = self.make_row(parent, "Starting time:")
        self.start_time = tkinter.StringVar(value=START_TIMES[0])
        tkinter.OptionMenu(row, self.start_time, *START_TIMES,
                           command=self.refresh_rooms_event).pack(side="left")

        row, label = self.make_row(parent, "Ending time:")
        self.end_time = tkinter.StringVar(value=END_TIMES[0])
        tkinter.OptionMenu(row, self.end_time, *END_TIMES,
                           command=self.refresh_rooms_event).pack(side="left")

        row, label = self.make_row(parent, "Purpose:")
        self.purpose_entry = tkinter.Entry(row, width=28)
        self.purpose_entry.pack(side="left")

    def show_role_fields(self):
        """Show or hide fields depending on Student / Teacher."""
        if self.role.get() == "Student":
            self.id_label.config(text="Student number:")
            self.program_label.config(text="Program:")
            self.student_frame.pack(fill="x")
        else:
            self.id_label.config(text="Teacher ID:")
            self.program_label.config(text="Program/Dept:")
            self.student_frame.pack_forget()

    
    def build_availability(self, parent):
        tkinter.Label(parent, text="Laboratory Availability",
                      font=("Arial", 11, "bold"), bg=BG_PANEL).pack(pady=4)
        tkinter.Label(parent, text="(uses the date and time chosen in the form)",
                      bg=BG_PANEL, font=("Arial", 8)).pack()

        row = tkinter.Frame(parent, bg=BG_PANEL)
        row.pack(fill="x", padx=8, pady=4)
        tkinter.Label(row, text="Floor:", bg=BG_PANEL).pack(side="left")
        self.floor = tkinter.StringVar(value="All")
        tkinter.OptionMenu(row, self.floor, "All", "1", "2", "3", "4", "5",
                           command=self.refresh_rooms_event).pack(side="left")

        row = tkinter.Frame(parent, bg=BG_PANEL)
        row.pack(fill="x", padx=8)
        self.show_mode = tkinter.StringVar(value="All")
        for m in ["All", "Available", "Occupied"]:
            tkinter.Radiobutton(row, text=m, variable=self.show_mode, value=m,
                                bg=BG_PANEL, command=self.refresh_rooms).pack(side="left")

        box = tkinter.Frame(parent, bg=BG_PANEL)
        box.pack(fill="both", expand=True, padx=8, pady=6)
        scroll = tkinter.Scrollbar(box)
        scroll.pack(side="right", fill="y")
        self.room_list = tkinter.Listbox(box, yscrollcommand=scroll.set, height=14)
        self.room_list.pack(side="left", fill="both", expand=True)
        scroll.config(command=self.room_list.yview)

        tkinter.Button(parent, text="Refresh List", width=16,
                       command=self.refresh_rooms).pack(pady=2)
        tkinter.Button(parent, text="Check Selected Room", width=16,
                       command=self.check_one_room).pack(pady=(2, 8))

    
    def read_schedule(self):
        """Returns (day, start, end) or None if something is wrong."""
        day_text = self.date_entry.get().strip()
        try:
            day_object = datetime.strptime(day_text, "%Y-%m-%d").date()
        except ValueError:
            messagebox.showerror("Error", "Invalid date. Use the format YYYY-MM-DD.")
            return None
        if day_object < date.today():
            messagebox.showerror("Error", "You cannot reserve a date that already passed.")
            return None
        day = day_object.strftime("%Y-%m-%d")

        start = HOUR_NUMBER.get(self.start_time.get())
        end = HOUR_NUMBER.get(self.end_time.get())
        if start is None or end is None:
            messagebox.showerror("Error", "Please choose a starting and ending time.")
            return None
        if start < OPEN_HOUR or end > CLOSE_HOUR:
            messagebox.showerror("Error", "Reservations are only from 7:00 AM to 8:00 PM.")
            return None
        if end <= start:
            messagebox.showerror("Error", "Ending time must be later than starting time.")
            return None
        return day, start, end

    def refresh_rooms_event(self, value):
        self.refresh_rooms()

    def refresh_rooms(self):
        """Fill the listbox with rooms for the chosen date and time."""
        self.room_list.delete(0, "end")
        # Check the date/time quietly (no popups while browsing)
        try:
            day = datetime.strptime(self.date_entry.get().strip(), "%Y-%m-%d").strftime("%Y-%m-%d")
        except ValueError:
            self.room_list.insert("end", "Enter a valid date (YYYY-MM-DD)")
            return
        start = HOUR_NUMBER.get(self.start_time.get())
        end = HOUR_NUMBER.get(self.end_time.get())
        if end <= start:
            self.room_list.insert("end", "Ending time must be after start")
            return

        last_floor = ""
        for room in ROOMS:
            floor = str(get_floor(room))
            if self.floor.get() != "All" and floor != self.floor.get():
                continue
            free = manager.is_available(room, day, start, end)
            if free:
                status = "Available"
            else:
                status = "Occupied"
            if self.show_mode.get() == "All" or self.show_mode.get() == status:
                if floor != last_floor:      # heading for each floor
                    self.room_list.insert("end", "--- Floor " + floor + " ---")
                    last_floor = floor
                self.room_list.insert("end", room + "  -  " + status)
        if self.room_list.size() == 0:
            self.room_list.insert("end", "(no rooms found)")

    def check_one_room(self):
        schedule = self.read_schedule()
        if schedule is None:
            return
        day, start, end = schedule
        room = self.room.get()
        choice = self.room_list.curselection()
        if len(choice) > 0:
            line = self.room_list.get(choice[0])
            if line.startswith("COE"):
                room = line[0:7]
                self.room.set(room)
        if manager.is_available(room, day, start, end):
            messagebox.showinfo("Availability", room + " is AVAILABLE on " + day +
                                " from " + hour_text(start) + " to " + hour_text(end) + ".")
        else:
            messagebox.showinfo("Availability", room + " is OCCUPIED during that time.")

    
    def reserve(self):
        name = self.name_entry.get().strip()
        person_id = self.id_entry.get().strip()
        program = self.program.get()
        purpose = self.purpose_entry.get().strip()
        role = self.role.get()

        if name == "" or person_id == "" or purpose == "" or program not in PROGRAMS:
            messagebox.showerror("Error", "Please complete all required fields.")
            return
        if not only_letters_and_spaces(name):
            messagebox.showerror("Error", "Name must contain letters only.")
            return
        if not valid_id_number(person_id):
            if role == "Student":
                messagebox.showerror("Error", "Invalid student number. Use digits and dashes (at least 5 digits).")
            else:
                messagebox.showerror("Error", "Invalid teacher ID. Use digits and dashes (at least 5 digits).")
            return

        year = ""
        section = ""
        if role == "Student":
            year = self.year.get()
            section = self.section.get()
            if year not in YEAR_LEVELS or section not in SECTIONS:
                messagebox.showerror("Error", "Invalid year level or section.")
                return

        schedule = self.read_schedule()
        if schedule is None:
            return
        day, start, end = schedule
        room = self.room.get()

        if not manager.is_available(room, day, start, end):
            messagebox.showerror("Error", "This laboratory is already reserved during the selected time.")
            return

        info = {"role": role, "name": name, "person_id": person_id,
                "program": program, "year": year, "section": section,
                "date": day, "room": room, "start": start, "end": end,
                "purpose": purpose}
        manager.add_reservation(info)
        messagebox.showinfo("Success", "Reservation successful!")
        self.refresh_rooms()

    def clear_form(self):
        self.name_entry.delete(0, "end")
        self.id_entry.delete(0, "end")
        self.purpose_entry.delete(0, "end")
        self.program.set("Select program")
        self.year.set(YEAR_LEVELS[0])
        self.section.set(SECTIONS[0])
        self.role.set("Student")
        self.room.set(ROOMS[0])
        self.start_time.set(START_TIMES[0])
        self.end_time.set(END_TIMES[0])
        self.date_entry.delete(0, "end")
        self.date_entry.insert(0, date.today().strftime("%Y-%m-%d"))
        self.floor.set("All")
        self.show_mode.set("All")
        self.show_role_fields()
        self.refresh_rooms()

    def open_admin_login(self):
        AdminLogin(self.window, self)



class AdminLogin:
    def __init__(self, parent, main_window):
        self.parent = parent
        self.main_window = main_window
        self.win = tkinter.Toplevel(parent)
        self.win.title("Administrator Login")
        self.win.geometry("300x180")
        self.win.configure(bg=BG)

        tkinter.Label(self.win, text="Administrator Login",
                      font=("Arial", 12, "bold"), bg=BG).pack(pady=8)
        row = tkinter.Frame(self.win, bg=BG)
        row.pack(pady=2)
        tkinter.Label(row, text="Username:", width=10, bg=BG).pack(side="left")
        self.user_entry = tkinter.Entry(row, width=18)
        self.user_entry.pack(side="left")
        row = tkinter.Frame(self.win, bg=BG)
        row.pack(pady=2)
        tkinter.Label(row, text="Password:", width=10, bg=BG).pack(side="left")
        self.pass_entry = tkinter.Entry(row, width=18, show="*")
        self.pass_entry.pack(side="left")
        tkinter.Button(self.win, text="Login", width=10,
                       command=self.check_login).pack(pady=10)

    def check_login(self):
        if (self.user_entry.get() == ADMIN_USERNAME and
                self.pass_entry.get() == ADMIN_PASSWORD):
            self.win.destroy()
            AdminWindow(self.parent, self.main_window)
        else:
            messagebox.showerror("Error", "Invalid administrator password.")
            self.pass_entry.delete(0, "end")



class AdminWindow:
    def __init__(self, parent, main_window):
        self.main_window = main_window
        self.shown = []   
        self.win = tkinter.Toplevel(parent)
        self.win.title("Administrator Panel")
        self.win.geometry("820x560")
        self.win.configure(bg=BG)

        tkinter.Label(self.win, text="Administrator Panel",
                      font=("Arial", 14, "bold"), bg=BG).pack(pady=6)

        
        top = tkinter.Frame(self.win, bg=BG_PANEL, bd=2, relief="groove")
        top.pack(fill="both", expand=True, padx=10, pady=4)
        tkinter.Label(top, text="Reservations", font=("Arial", 11, "bold"),
                      bg=BG_PANEL).pack(pady=2)

        row = tkinter.Frame(top, bg=BG_PANEL)
        row.pack(fill="x", padx=8)
        tkinter.Label(row, text="Search (name, room, date, program):",
                      bg=BG_PANEL).pack(side="left")
        self.search_entry = tkinter.Entry(row, width=22)
        self.search_entry.pack(side="left", padx=4)
        tkinter.Button(row, text="Search", command=self.show_reservations).pack(side="left", padx=2)
        tkinter.Button(row, text="Show All", command=self.show_all).pack(side="left", padx=2)

        box = tkinter.Frame(top, bg=BG_PANEL)
        box.pack(fill="both", expand=True, padx=8, pady=4)
        scroll = tkinter.Scrollbar(box)
        scroll.pack(side="right", fill="y")
        self.res_list = tkinter.Listbox(box, yscrollcommand=scroll.set, height=8)
        self.res_list.pack(side="left", fill="both", expand=True)
        scroll.config(command=self.res_list.yview)

        row = tkinter.Frame(top, bg=BG_PANEL)
        row.pack(pady=4)
        tkinter.Button(row, text="View Details", width=14,
                       command=self.view_details).pack(side="left", padx=3)
        tkinter.Button(row, text="Cancel Reservation", width=18,
                       command=self.cancel_selected).pack(side="left", padx=3)

        
        bottom = tkinter.Frame(self.win, bg=BG_PANEL, bd=2, relief="groove")
        bottom.pack(fill="both", expand=True, padx=10, pady=4)
        tkinter.Label(bottom, text="Laboratories (by date)", font=("Arial", 11, "bold"),
                      bg=BG_PANEL).pack(pady=2)

        row = tkinter.Frame(bottom, bg=BG_PANEL)
        row.pack(fill="x", padx=8)
        tkinter.Label(row, text="Date (YYYY-MM-DD):", bg=BG_PANEL).pack(side="left")
        self.date_entry = tkinter.Entry(row, width=12)
        self.date_entry.insert(0, date.today().strftime("%Y-%m-%d"))
        self.date_entry.pack(side="left", padx=4)
        tkinter.Button(row, text="All Labs", command=self.show_all_labs).pack(side="left", padx=2)
        tkinter.Button(row, text="Vacant Labs", command=self.show_vacant_labs).pack(side="left", padx=2)
        tkinter.Button(row, text="Occupied Labs", command=self.show_occupied_labs).pack(side="left", padx=2)

        box = tkinter.Frame(bottom, bg=BG_PANEL)
        box.pack(fill="both", expand=True, padx=8, pady=4)
        scroll2 = tkinter.Scrollbar(box)
        scroll2.pack(side="right", fill="y")
        self.lab_list = tkinter.Listbox(box, yscrollcommand=scroll2.set, height=8)
        self.lab_list.pack(side="left", fill="both", expand=True)
        scroll2.config(command=self.lab_list.yview)

        tkinter.Button(self.win, text="Logout", width=10,
                       command=self.win.destroy).pack(pady=6)

        self.show_reservations()

    
    def show_all(self):
        self.search_entry.delete(0, "end")
        self.show_reservations()

    def show_reservations(self):
        word = self.search_entry.get().strip().lower()
        self.res_list.delete(0, "end")
        self.shown = []
        for r in manager.reservations:
            text = (r["name"] + " " + r["room"] + " " + r["date"] + " " + r["program"]).lower()
            if word == "" or word in text:
                self.shown.append(r)
                self.res_list.insert("end", reservation_line(r))
        if len(self.shown) == 0:
            self.res_list.insert("end", "(no reservations found)")

    def get_selected(self):
        choice = self.res_list.curselection()
        if len(choice) == 0 or len(self.shown) == 0:
            messagebox.showwarning("Select", "Please select a reservation first.")
            return None
        return self.shown[choice[0]]

    def view_details(self):
        r = self.get_selected()
        if r is None:
            return
        text = ("Reservation ID: " + str(r["id"]) + "\n" +
                "Type: " + r["role"] + "\n" +
                "Name: " + r["name"] + "\n" +
                "ID Number: " + r["person_id"] + "\n" +
                "Program: " + r["program"] + "\n")
        if r["role"] == "Student":
            text = text + "Year/Section: " + r["year"] + " - " + r["section"] + "\n"
        text = (text + "Date: " + r["date"] + "\n" +
                "Room: " + r["room"] + "\n" +
                "Time: " + hour_text(r["start"]) + " to " + hour_text(r["end"]) + "\n" +
                "Purpose: " + r["purpose"])
        messagebox.showinfo("Reservation Details", text)

    def cancel_selected(self):
        r = self.get_selected()
        if r is None:
            return
        if messagebox.askyesno("Cancel", "Cancel reservation " + str(r["id"]) + " for " + r["room"] + "?"):
            manager.cancel_reservation(r["id"])
            messagebox.showinfo("Cancelled", "Reservation cancelled.")
            self.show_reservations()
            self.main_window.refresh_rooms()

    
    def show_labs(self, mode):
        day_text = self.date_entry.get().strip()
        try:
            day = datetime.strptime(day_text, "%Y-%m-%d").strftime("%Y-%m-%d")
        except ValueError:
            messagebox.showerror("Error", "Invalid date. Use the format YYYY-MM-DD.")
            return
        self.lab_list.delete(0, "end")
        for room in ROOMS:
            count = len(manager.reservations_for_room(room, day))
            if count == 0:
                status = "Vacant"
            else:
                status = "Occupied (" + str(count) + " reservation/s)"
            if mode == "All" or (mode == "Vacant" and count == 0) or (mode == "Occupied" and count > 0):
                self.lab_list.insert("end", room + "  -  " + status)

    def show_all_labs(self):
        self.show_labs("All")

    def show_vacant_labs(self):
        self.show_labs("Vacant")

    def show_occupied_labs(self):
        self.show_labs("Occupied")


root = tkinter.Tk()
app = MainWindow(root)
root.mainloop()
