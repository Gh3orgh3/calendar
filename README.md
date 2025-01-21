import calendar
import json
from tkinter import *
from tkinter import ttk
from tkinter import simpledialog

# Dictionar pentru stocarea evenimentelor (cu suport pentru multiple evenimente pe zi)
events = {}

# Numele fișierului pentru salvarea evenimentelor
EVENTS_FILE = "evenimente.json"

def save_events():
    """Salveaza evenimentele într-un fișier JSON."""
    try:
        with open(EVENTS_FILE, "w") as file:
            json.dump({f"{year}-{month}-{day}": event_list for (year, month, day), event_list in events.items()}, file)
        lbl_status.config(text="Evenimentele au fost salvate cu succes!", fg="green")
    except Exception as e:
        lbl_status.config(text=f"Eroare la salvarea evenimentelor: {str(e)}", fg="red")

def load_events():
    """Incarcă evenimentele salvate din fisieruu JSON."""
    global events
    try:
        with open(EVENTS_FILE, "r") as file:
            data = json.load(file)
            # Convertim cheile string în tuple
            events = {tuple(map(int, k.split("-"))): v for k, v in data.items()}
    except (FileNotFoundError, json.JSONDecodeError):
        events = {}


def delete_event_specific(year, month, day, event):
    """sterge un eveniment specific."""
    if (year, month, day) in events:
        events[(year, month, day)].remove(event)
        if not events[(year, month, day)]:  # Elimină cheia dacă lista devine goală
            del events[(year, month, day)]
        lbl_status.config(text=f"Eveniment șters: {event}", fg="green")

def view_events():
    """Afiseaza evenimentele intr-o fereastră separata."""
    event_window = Toplevel(root)
    event_window.title("Evenimente")
    event_window.geometry("500x500")
    event_window.config(bg="#ffffff")
    
    Label(event_window, text="Evenimente", font=("Arial", 14, "bold"), bg="#ffffff").pack(pady=10)
    
    if not events:
        Label(event_window, text="Nu există evenimente adăugate!", font=("Arial", 12), bg="#ffffff").pack(pady=10)
    else:
        for (year, month, day), event_list in sorted(events.items()):
            Label(
                event_window, text=f"{day}/{month}/{year}:", font=("Arial", 12, "bold"), bg="#ffffff"
            ).pack(anchor="w", padx=20, pady=5)
            for event in event_list:
                event_frame = Frame(event_window, bg="#f0f0f0")
                event_frame.pack(fill="x", padx=30, pady=2, anchor="w")
                Label(event_frame, text=f"- {event}", font=("Arial", 12), bg="#f0f0f0").pack(side="left", padx=10)
                Button(
                    event_frame, text="Șterge", font=("Arial", 8), bg="red", fg="white",
                    command=lambda y=year, m=month, d=day, e=event: (
                        delete_event_specific(y, m, d, e), event_window.destroy(), view_events()
                    )
                ).pack(side="right", padx=5)

def add_event(year, month, day):
    """Adaugă un eveniment în dicționar."""
    event = simpledialog.askstring("Adaugă eveniment", f"Eveniment pentru {day}/{month}/{year}:")
    if event:
        events.setdefault((year, month, day), []).append(event)  # Adaugă în listă
        lbl_status.config(text=f"Eveniment adăugat: {day}/{month}/{year} - {event}", fg="green")
        show_calendar()
    else:
        lbl_status.config(text="Niciun eveniment adăugat.", fg="orange")

def show_calendar():
    """Afișează calendarul pentru anul și luna selectată."""
    for widget in calendar_frame.winfo_children():
        widget.destroy()
    
    try:
        year = int(entry_year.get())
        month = combo_month.current() + 1
        
        if year < 1 or month < 1 or month > 12:
            raise ValueError
        
        cal = calendar.monthcalendar(year, month)
        days = ["Luni", "Marți", "Miercuri", "Joi", "Vineri", "Sâmbătă", "Duminică"]
        
        for col, day in enumerate(days):
            lbl_day = Label(
                calendar_frame, text=day, font=("Arial", 10, "bold"),
                bg="#f0f0f0", fg="#000000", relief="ridge"
            )
            lbl_day.grid(row=0, column=col, sticky="nsew")
            calendar_frame.columnconfigure(col, weight=1)
        
        for row_idx, week in enumerate(cal):
            for col_idx, day in enumerate(week):
                if day != 0:
                    day_color = "#e6e6e6"
                    if (year, month, day) in events:
                        day_color = "#add8e6"
                    frame_day = Frame(calendar_frame, bg=day_color, relief="groove", bd=2)
                    frame_day.grid(row=row_idx + 1, column=col_idx, sticky="nsew", padx=2, pady=2)
                    
                    btn_day = Button(
                        frame_day, text=str(day), font=("Arial", 10),
                        bg=day_color, relief="flat",
                        command=lambda d=day: add_event(year, month, d)
                    )
                    btn_day.pack(fill="both", expand=True)

        for row in range(len(cal) + 1):
            calendar_frame.rowconfigure(row, weight=1)

    except ValueError:
        lbl_status.config(text="Eroare: Introdu un an și o lună valide!", fg="red")

# Incarca evenimentele la pornirea aplicației
load_events()

# Fereastra in care o sa fie aplicatia
root = Tk()
root.title("Calendar Interactiv")
root.geometry("1024x768")
root.config(bg="#f0f0f0")

Label(root, text="Calendar Interactiv", font=("Arial", 16, "bold"), bg="#f0f0f0", fg="#000000").pack(pady=10)

frame_inputs = Frame(root, bg="#f0f0f0")
frame_inputs.pack(pady=10)

Label(frame_inputs, text="Introdu anul:", font=("Arial", 12), bg="#f0f0f0", fg="#000000").grid(row=0, column=0, padx=10, pady=5)
entry_year = Entry(frame_inputs, width=10, font=("Arial", 12))
entry_year.grid(row=0, column=1, padx=10, pady=5)

Label(frame_inputs, text="Alege luna:", font=("Arial", 12), bg="#f0f0f0", fg="#000000").grid(row=0, column=2, padx=10, pady=5)
combo_month = ttk.Combobox(frame_inputs, values=[
    "Ianuarie", "Februarie", "Martie", "Aprilie", "Mai", "Iunie", 
    "Iulie", "August", "Septembrie", "Octombrie", "Noiembrie", "Decembrie"
], width=12, state="readonly")
combo_month.grid(row=0, column=3, padx=10, pady=5)
combo_month.current(0)
    
button_show = ttk.Button(frame_inputs, text="Afișează calendarul", command=show_calendar)
button_show.grid(row=0, column=4, padx=10, pady=5)

button_events = ttk.Button(frame_inputs, text="Vezi evenimente", command=view_events)
button_events.grid(row=0, column=5, padx=10, pady=5)

button_save = ttk.Button(frame_inputs, text="Salvează evenimentele", command=save_events)
button_save.grid(row=0, column=6, padx=10, pady=5)

lbl_status = Label(root, text="", bg="#f0f0f0", font=("Arial", 10), fg="#000000")
lbl_status.pack(pady=5)

calendar_frame = Frame(root, bg="#f0f0f0")
calendar_frame.pack(expand=True, fill="both", pady=20)

root.mainloop()
