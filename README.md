import tkinter as tk
from tkinter import ttk
import re

COMMON_PASSWORDS = {
    "123456", "password", "123456789", "qwerty", "abc123",
    "111111", "12345678", "password1", "12345", "123123"
}

# ---------------- Password Strength Logic ----------------
def check_password_strength(password):
    score = 0
    feedback = []

    if len(password) >= 12:
        score += 2
    elif len(password) >= 8:
        score += 1
    else:
        feedback.append("Use at least 12 characters.")

    if re.search(r"[a-z]", password):
        score += 1
    else:
        feedback.append("Add lowercase letters [a-z].")

    if re.search(r"[A-Z]", password):
        score += 1
    else:
        feedback.append("Add uppercase letters [A-Z].")

    if re.search(r"[0-9]", password):
        score += 1
    else:
        feedback.append("Add numbers [0-9].")

    if re.search(r"[!@#$%^&*(),.?\":{}|<>]", password):
        score += 1
    else:
        feedback.append("Add special characters.")

    if password.lower() in COMMON_PASSWORDS:
        score = 0
        feedback.append("This password is too common!")

    return score, feedback


# ---------------- Update GUI ----------------
def update_strength(event=None):
    password = entry.get()
    score, suggestions = check_password_strength(password)

    # Progress bar value (0–6) ->
    strength_bar["value"] = score

    # Color changes
    if score >= 6:
        color = "#2ecc71"   
        text = "Very Strong"
    elif score == 5:
        color = "#27ae60"
        text = "Strong"
    elif score in (3, 4):
        color = "#f1c40f"
        text = "Medium"
    elif score in (1, 2):
        color = "#e67e22"
        text = "Weak"
    else:
        color = "#e74c3c"
        text = "Very Weak"

    strength_label.config(text=text, foreground=color)
    style.configure("TProgressbar", troughcolor="#eee", background=color)

    # Update suggestion box ->
    suggestion_box.config(state="normal")
    suggestion_box.delete(1.0, tk.END)
    if suggestions:
        for s in suggestions:
            suggestion_box.insert(tk.END, f"• {s}\n")
    else:
        suggestion_box.insert(tk.END, "Looks good!")
    suggestion_box.config(state="disabled")


# ---------------- Toggle Password Visibility ----------------
def toggle_password():
    if entry["show"] == "":
        entry.config(show="*")
        toggle_btn.config(text="Show")
    else:
        entry.config(show="")
        toggle_btn.config(text="Hide")


# ---------------- GUI Setup ----------------
root = tk.Tk()
root.title("Password Strength Checker")
root.geometry("480x420")
root.resizable(False, False)

# Modern fonts ->
TITLE_FONT = ("Segoe UI", 18, "bold")
LABEL_FONT = ("Segoe UI", 12)
TEXT_FONT = ("Segoe UI", 10)

# Custom style ->
style = ttk.Style()
style.theme_use("default")
style.configure("TEntry", padding=5)
style.configure("TButton", font=LABEL_FONT, padding=5)
style.configure("TProgressbar", thickness=15, troughcolor="#ddd", background="#3498db")

# Header gradient-like frame ->
header = tk.Frame(root, height=70, bg="#4b79a1")
header.pack(fill="x")

title = tk.Label(header, text="Password Strength Checker",
                 bg="#4b79a1", fg="white", font=TITLE_FONT)
title.place(relx=0.5, rely=0.5, anchor="center")

# Main content frame ->
content = tk.Frame(root, padx=20, pady=20)
content.pack(fill="both")

tk.Label(content, text="Enter Password:", font=LABEL_FONT).pack(anchor="w")

entry_frame = tk.Frame(content)
entry_frame.pack(fill="x")

entry = ttk.Entry(entry_frame, show="*", width=30)
entry.grid(row=0, column=0, padx=5, pady=10)
entry.bind("<KeyRelease>", update_strength)

toggle_btn = ttk.Button(entry_frame, text="Show", command=toggle_password)
toggle_btn.grid(row=0, column=1, padx=5)

strength_label = tk.Label(content, text="Strength Meter", font=("Segoe UI", 12, "bold"))
strength_label.pack(pady=5)

strength_bar = ttk.Progressbar(content, maximum=6, length=350)
strength_bar.pack(pady=10)

tk.Label(content, text="Suggestions:", font=LABEL_FONT).pack(anchor="w", pady=(10, 0))

suggestion_box = tk.Text(content, height=6, width=50, font=TEXT_FONT,
                         state="disabled", bg="#f7f7f7", relief="flat")
suggestion_box.pack(pady=5)

root.mainloop()
