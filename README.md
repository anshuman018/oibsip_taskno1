# -_-1 import tkinter as tk
from tkinter import ttk
import sqlite3
import matplotlib.pyplot as plt
from matplotlib.backends.backend_tkagg import FigureCanvasTkAgg


class BodyMassIndexCalculator:
    def __init__(self, master):
        self.master = master
        self.master.title("BMI Calculator")
        self.master.geometry("400x400")

        # Variables for user input
        self.weight_entry_var = tk.DoubleVar()
        self.height_entry_var = tk.DoubleVar()
        self.result_var = tk.StringVar()

        # Create and place widgets
        self.create_gui_elements()

        # SQLite database setup
        self.database_connection = sqlite3.connect("bmi_data.db")
        self.create_database_table()

    def create_gui_elements(self):
        # Labels
        ttk.Label(self.master, text="Weight (kg):").grid(row=0, column=0, padx=10, pady=10)
        ttk.Label(self.master, text="Height (m):").grid(row=1, column=0, padx=10, pady=10)
        ttk.Label(self.master, text="BMI Result:").grid(row=3, column=0, padx=10, pady=10)

        # Entry widgets
        ttk.Entry(self.master, textvariable=self.weight_entry_var).grid(row=0, column=1, padx=10, pady=10)
        ttk.Entry(self.master, textvariable=self.height_entry_var).grid(row=1, column=1, padx=10, pady=10)

        # Calculate button
        ttk.Button(self.master, text="Calculate BMI", command=self.calculate_bmi).grid(row=2, column=0, columnspan=2, pady=10)

        # Result label
        ttk.Label(self.master, textvariable=self.result_var).grid(row=3, column=1, padx=10, pady=10)

        # Save button
        ttk.Button(self.master, text="Save Data", command=self.save_data).grid(row=4, column=0, columnspan=2, pady=10)

        # View Data button
        ttk.Button(self.master, text="View Data", command=self.view_data).grid(row=5, column=0, columnspan=2, pady=10)

    def calculate_bmi(self):
        try:
            weight = self.weight_entry_var.get()
            height = self.height_entry_var.get()

            if weight <= 0 or height <= 0:
                raise ValueError("Weight and height must be positive values.")

            bmi = weight / (height ** 2)
            result_message = f"BMI: {bmi:.2f}, {self.categorize_bmi(bmi)}"
            self.result_var.set(result_message)

        except ValueError as error_message:
            self.result_var.set(str(error_message))

    def categorize_bmi(self, bmi):
        if bmi < 18.5:
            return "Underweight"
        elif 18.5 <= bmi < 24.9:
            return "Normal Weight"
        elif 25 <= bmi < 29.9:
            return "Overweight"
        else:
            return "Obese"

    def create_database_table(self):
        cursor = self.database_connection.cursor()
        cursor.execute(
            "CREATE TABLE IF NOT EXISTS bmi_data (id INTEGER PRIMARY KEY AUTOINCREMENT, weight REAL, height REAL, bmi REAL)"
        )
        self.database_connection.commit()

    def save_data(self):
        try:
            weight = self.weight_entry_var.get()
            height = self.height_entry_var.get()
            bmi = weight / (height ** 2)

            cursor = self.database_connection.cursor()
            cursor.execute("INSERT INTO bmi_data (weight, height, bmi) VALUES (?, ?, ?)", (weight, height, bmi))
            self.database_connection.commit()

        except Exception as error:
            print(f"Error saving data: {error}")

    def view_data(self):
        cursor = self.database_connection.cursor()
        cursor.execute("SELECT weight, height, bmi FROM bmi_data")

        data = cursor.fetchall()

        if not data:
            print("No data available.")
            return

        weights = [item[0] for item in data]
        heights = [item[1] for item in data]
        bmis = [item[2] for item in data]

        # Data visualization using Matplotlib
        fig, (ax1, ax2) = plt.subplots(2, 1, figsize=(6, 6))
        ax1.plot(weights, label="Weight")
        ax1.plot(heights, label="Height")
        ax1.set_ylabel("Value")
        ax1.legend()

        ax2.plot(bmis, color="red", label="BMI")
        ax2.set_xlabel("Data Entry")
        ax2.set_ylabel("BMI")
        ax2.legend()

        plt.show()


def main():
    root = tk.Tk()
    app = BodyMassIndexCalculator(root)
    root.mainloop()


if __name__ == "__main__":
    main()
