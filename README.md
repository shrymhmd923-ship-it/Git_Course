# Git_Course
For shrymhmd Git Course

import tkinter as tk
from tkinter import messagebox
import sqlite3
import csv

# ================= DATABASE =================
class DB:
    def __init__(self):
        self.conn = sqlite3.connect("business_pro.db")
        self.cur = self.conn.cursor()
        self.cur.execute("""
            CREATE TABLE IF NOT EXISTS orders (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                customer TEXT,
                product TEXT,
                price REAL
            )
        """)
        self.conn.commit()

    def add(self, c, p, pr):
        self.cur.execute("INSERT INTO orders (customer, product, price) VALUES (?, ?, ?)", (c, p, pr))
        self.conn.commit()

    def delete(self, id_):
        self.cur.execute("DELETE FROM orders WHERE id=?", (id_,))
        self.conn.commit()

    def fetch(self):
        self.cur.execute("SELECT * FROM orders")
        return self.cur.fetchall()

    def update(self, id_, c, p, pr):
        self.cur.execute("""
            UPDATE orders
            SET customer=?, product=?, price=?
            WHERE id=?
        """, (c, p, pr, id_))
        self.conn.commit()


# ================= APP =================
class App:
    def __init__(self, root):
        self.db = DB()
        self.root = root
        self.root.title("Smart Business Assistant PRO")
        self.root.geometry("600x650")

        # Inputs
        tk.Label(root, text="ID (for update/delete)").pack()
        self.id_entry = tk.Entry(root)
        self.id_entry.pack()

        tk.Label(root, text="Customer").pack()
        self.c_entry = tk.Entry(root)
        self.c_entry.pack()

        tk.Label(root, text="Product").pack()
        self.p_entry = tk.Entry(root)
        self.p_entry.pack()

        tk.Label(root, text="Price").pack()
        self.pr_entry = tk.Entry(root)
        self.pr_entry.pack()

        # Buttons
        tk.Button(root, text="Add Order", command=self.add).pack(pady=2)
        tk.Button(root, text="Update Order", command=self.update).pack(pady=2)
        tk.Button(root, text="Delete Order", command=self.delete).pack(pady=2)
        tk.Button(root, text="Export CSV", command=self.export).pack(pady=2)
        tk.Button(root, text="Refresh", command=self.load).pack(pady=2)

        self.listbox = tk.Listbox(root, width=80, height=20)
        self.listbox.pack()

        self.total = tk.Label(root, text="Total: 0")
        self.total.pack()

        self.load()

    def add(self):
        try:
            c = self.c_entry.get()
            p = self.p_entry.get()
            pr = float(self.pr_entry.get())

            self.db.add(c, p, pr)
            self.clear()
            self.load()
        except:
            messagebox.showerror("Error", "Invalid input")

    def delete(self):
        try:
            id_ = int(self.id_entry.get())
            self.db.delete(id_)
            self.load()
        except:
            messagebox.showerror("Error", "Enter valid ID")

    def update(self):
        try:
            id_ = int(self.id_entry.get())
            c = self.c_entry.get()
            p = self.p_entry.get()
            pr = float(self.pr_entry.get())

            self.db.update(id_, c, p, pr)
            self.load()
        except:
            messagebox.showerror("Error", "Invalid input")

    def export(self):
        data = self.db.fetch()

        with open("report.csv", "w", newline="") as f:
            writer = csv.writer(f)
            writer.writerow(["ID", "Customer", "Product", "Price"])
            writer.writerows(data)

        messagebox.showinfo("Done", "Exported to report.csv")

    def load(self):
        self.listbox.delete(0, tk.END)
        data = self.db.fetch()

        total = 0
        for row in data:
            self.listbox.insert(tk.END, f"ID:{row[0]} | {row[1]} | {row[2]} | {row[3]}")
            total += row[3]

        self.total.config(text=f"Total Sales: {total}")

    def clear(self):
        self.c_entry.delete(0, tk.END)
        self.p_entry.delete(0, tk.END)
        self.pr_entry.delete(0, tk.END)


root = tk.Tk()
App(root)
root.mainloop()
