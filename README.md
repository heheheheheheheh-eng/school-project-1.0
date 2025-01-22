# school-project-1.0
import pandas as pd
import os
import warnings

# Suppress FutureWarnings
warnings.simplefilter(action='ignore', category=FutureWarning)

# File Paths
BOOKS_FILE = "C:\\Users\\ks936\\OneDrive\\Desktop\\AM BOOKSELLERS.csv"
SALES_FILE = "sales.csv"

# Initialize files if they don't exist
if not os.path.exists(BOOKS_FILE):
    pd.DataFrame(columns=["BOOKID", "BOOK NAME", "QUANTITY", "PRICE"]).to_csv(BOOKS_FILE, index=False)

if not os.path.exists(SALES_FILE):
    pd.DataFrame(columns=["BOOKID", "BOOK NAME", "QUANTITY SOLD", "TOTAL PRICE"]).to_csv(SALES_FILE, index=False)

def safe_input(prompt, dtype):
    """Helper function to handle safe input conversions."""
    while True:
        try:
            return dtype(input(prompt))
        except ValueError:
            print("Invalid input. Please try again.")

def add_books():
    """Add new books to the inventory."""
    try:
        df = pd.read_csv(BOOKS_FILE)
        num_entries = safe_input("Enter number of books to add: ", int)
        for _ in range(num_entries):
            book_id = input("Enter book ID: ").strip()
            book_name = input("Enter book name: ").strip()
            quantity = safe_input("Enter quantity: ", int)
            price = safe_input("Enter price: ", float)

            # Append new row using concat
            new_row = {"BOOKID": book_id, "BOOK NAME": book_name, "QUANTITY": quantity, "PRICE": price}
            df = pd.concat([df, pd.DataFrame([new_row])], ignore_index=True)

        # Write back to CSV
        df.to_csv(BOOKS_FILE, index=False)
        print("\nBooks added successfully!")
    except Exception as e:
        print(f"Error: {e}")

def view_books():
    """View available books."""
    try:
        df = pd.read_csv(BOOKS_FILE)
        if df.empty:
            print("\nNo books available in inventory.")
        else:
            print("\nAvailable Books:")
            print(df)
    except Exception as e:
        print(f"Error: {e}")

def purchase_books():
    """Handle book purchases."""
    try:
        books_df = pd.read_csv(BOOKS_FILE)
        if books_df.empty:
            print("\nNo books available for purchase.")
            return

        sales_df = pd.read_csv(SALES_FILE)
        print("\nAvailable Books:")
        print(books_df)

        num_books = safe_input("\nEnter number of books to purchase: ", int)
        total_price = 0
        purchased_books = []

        for _ in range(num_books):
            book_id = input("\nEnter book ID: ").strip()
            if book_id not in books_df["BOOKID"].values:
                print("Invalid book ID. Try again.")
                continue

            book_info = books_df[books_df["BOOKID"] == book_id].iloc[0]
            book_name = book_info["BOOK NAME"]
            available_qty = book_info["QUANTITY"]
            price_per_unit = book_info["PRICE"]

            print(f"Book Name: {book_name}, Available Quantity: {available_qty}, Price per unit: {price_per_unit}")
            quantity = safe_input("Enter quantity to purchase: ", int)

            if quantity > available_qty:
                print("Insufficient stock. Try again.")
                continue

            # Update inventory
            books_df.loc[books_df["BOOKID"] == book_id, "QUANTITY"] -= quantity
            # Calculate price
            price = quantity * price_per_unit
            total_price += price

            # Record sales
            purchased_books.append({"BOOKID": book_id, "BOOK NAME": book_name, "QUANTITY SOLD": quantity, "TOTAL PRICE": price})

        # Update files
        books_df.to_csv(BOOKS_FILE, index=False)
        sales_df = pd.concat([sales_df, pd.DataFrame(purchased_books)], ignore_index=True)
        sales_df.to_csv(SALES_FILE, index=False)

        # Display bill
        print("\n--------- A.M. BOOK SELLERS --------")
        print("--------------- BILL ---------------")
        print(pd.DataFrame(purchased_books))
        print(f"TOTAL PRICE: {total_price}")
    except Exception as e:
        print(f"Error: {e}")

def view_sales():
    """View sales records."""
    try:
        sales_df = pd.read_csv(SALES_FILE)
        if sales_df.empty:
            print("\nNo sales records available.")
        else:
            print("\nSales Records:")
            print(sales_df)
    except Exception as e:
        print(f"Error: {e}")

def main_menu():
    """Main Menu."""
    while True:
        print("\n--------------------------------------")
        print("            A.M. BOOK SELLERS         ")
        print("--------------------------------------")
        print("1: ADMIN")
        print("2: PURCHASE MENU")
        print("3: VIEW SALES")
        print("4: EXIT")
        print("--------------------------------------")
        choice = input("Enter your choice: ").strip()

        if choice == '1':
            while True:
                print("\n-------- ADMIN MENU --------")
                print("1: Add a New Book")
                print("2: View Data")
                print("3: Exit")
                admin_choice = input("Enter your choice: ").strip()

                if admin_choice == '1':
                    add_books()
                elif admin_choice == '2':
                    view_books()
                elif admin_choice == '3':
                    break
                else:
                    print("Invalid choice. Try again.")
        
        elif choice == '2':
            purchase_books()
        
        elif choice == '3':
            view_sales()
        
        elif choice == '4':
            print("Exited Successfully.")
            break

        else:
            print("Invalid choice. Try again.")

# Start the application
main_menu()
