import tkinter as tk
from tkinter import messagebox, ttk, filedialog
import sqlite3
from datetime import datetime
import csv
import os

# --- 1. Configuration de la base de données (Database Setup) ---
DB_NAME = 'transactions.db'
# Définition des entêtes pour le CSV dans l'ordre de la BD
COL_NAMES = ["id", "Billet", "Montant", "N_PC", "N_Compte", "Date", "Type"] 

def setup_database():
    """Crée la base de données et la table avec la structure finale."""
    conn = sqlite3.connect(DB_NAME)
    cursor = conn.cursor()
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS transactions (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            Billet TEXT NOT NULL,
            Montant REAL NOT NULL,
            N_PC TEXT,
            N_Compte TEXT,  
            Date TEXT,
            Type TEXT       -- Recette ou Dépense
        )
    ''')
    conn.commit()
    conn.close()

# --- 2. Fonctions de la base de données (Database Functions) ---

def insert_transaction(billet, montant, n_pc, n_compte, date, type_op):
    """Insère une nouvelle transaction. Correction: l'ordre des paramètres est critique."""
    conn = sqlite3.connect(DB_NAME)
    cursor = conn.cursor()
    
    try:
        # L'ordre ici doit correspondre strictement à l'ordre des colonnes dans la création de table
        cursor.execute('''
            INSERT INTO transactions (Billet, Montant, N_PC, N_Compte, Date, Type)
            VALUES (?, ?, ?, ?, ?, ?)
        ''', (billet, montant, n_pc, n_compte, date, type_op))
        conn.commit()
        messagebox.showinfo("Succès", "Transaction enregistrée avec succès!")
    except Exception as e:
        messagebox.showerror("Erreur", f"Une erreur est survenue lors de l'enregistrement: {e}")
    finally:
        conn.close()
        
def get_all_transactions():
    """Récupère toutes les transactions (l'ordre des colonnes est important)."""
    conn = sqlite3.connect(DB_NAME)
    cursor = conn.cursor()
    cursor.execute('SELECT * FROM transactions ORDER BY id DESC')
    records = cursor.fetchall()
    conn.close()
    return records

def get_all_transactions_for_export():
    """Récupère toutes les transactions avec les entêtes pour l'export."""
    conn = sqlite3.connect(DB_NAME)
    cursor = conn.cursor()
    cursor.execute('SELECT * FROM transactions ORDER BY id ASC')
    records = cursor.fetchall()
    conn.close()
    return COL_NAMES, records # Utilisation des noms de colonnes définis en haut du script

# --- 3. Interface de l'Application (Tkinter Interface) ---

class TransactionApp:
    def _init_(self, master):
        self.master = master
        master.title("Registre des Transactions - MAD 🇲🇦")
        master.geometry("950x700")

        setup_database()

        self.main_frame = tk.Frame(master, padx=10, pady=10)
        self.main_frame.pack(fill='both', expand=True)

        self.create_input_section()
        self.create_toolbar_section()
        self.create_view_section()
        self.load_transactions()

    def create_input_section(self):
        """Crée la section de saisie des nouvelles transactions."""
        input_frame = tk.LabelFrame(self.main_frame, text="Nouvelle Transaction", padx=10, pady=10)
        input_frame.pack(fill='x', pady=10)

        # Les champs mis à jour avec MAD
        labels = ["Billet (Description):", "Montant (MAD):", "N°PC (Réf. Document):", 
                  "N° Compte (N° Compte):", "Date (AAAA-MM-JJ):", "Type (Recette/Dépense):"]
        self.entries = {}
        
        types_options = ["Recette", "Dépense"]
        
        for i, label_text in enumerate(labels):
            label = tk.Label(input_frame, text=label_text)
            label.grid(row=i, column=0, padx=5, pady=5, sticky='w')
            
            if "Type" in label_text:
                self.type_var = tk.StringVar(input_frame)
                self.type_var.set(types_options[0])
                entry = ttk.Combobox(input_frame, textvariable=self.type_var, values=types_options, state="readonly")
            elif "Date" in label_text:
                entry = tk.Entry(input_frame, width=40)
                entry.insert(0, datetime.now().strftime("%Y-%m-%d"))
            else:
                entry = tk.Entry(input_frame, width=40)
            
            entry.grid(row=i, column=1, padx=5, pady=5, sticky='ew')
            self.entries[label_text.split('(')[0].strip()] = entry

        save_button = tk.Button(input_frame, text="✅ Enregistrer la Transaction", command=self.save_entry)
        save_button.grid(row=len(labels), column=0, columnspan=2, pady=10)
        
        input_frame.grid_columnconfigure(1, weight=1)
        
    def create_toolbar_section(self):
        """Crée la barre d'outils avec les boutons d'export et d'impression."""
        toolbar_frame = tk.Frame(self.main_frame)
        toolbar_frame.pack(fill='x', pady=5)
        
        export_button = tk.Button(toolbar_frame, text="⬇ Exporter (CSV)", command=self.export_to_csv)
        export_button.pack(side=tk.LEFT, padx=5)

        print_button = tk.Button(toolbar_frame, text="🖨 Imprimer", command=self.print_records)
        print_button.pack(side=tk.LEFT, padx=5)


    def create_view_section(self):
        """Crée la section d'affichage des enregistrements."""
        view_frame = tk.LabelFrame(self.main_frame, text="Historique des Transactions", padx=5, pady=5)
        view_frame.pack(fill='both', expand=True, pady=10)

        # Les colonnes pour l'affichage
        columns = ("#1", "#2", "#3", "#4", "#5", "#6", "#7")
        self.tree = ttk.Treeview(view_frame, columns=columns, show='headings')
        
        # Définition des entêtes de colonnes (Ordre de la BD: ID, Billet, Montant, N_PC, N_Compte, Date, Type)
        # Ordre d'affichage: ID, Date, Billet, Montant, N°PC, N° Compte, Type
        headings = ["ID", "Date", "Billet", "Montant (MAD)", "N°PC", "N° Compte", "Type"]
        widths = [40, 90, 180, 110, 100, 100, 90]
        
        for i, heading in enumerate(headings):
            self.tree.heading(f
