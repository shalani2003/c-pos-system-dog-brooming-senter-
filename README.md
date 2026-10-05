# Pawsome Pet POS (C# Windows Forms + SQLite)

A desktop Point of Sale application for a **pet supplies shop**, built with
C# / .NET 8 Windows Forms and a local **SQLite** database
(`PetShopPOS.db`). No server setup, no internet connection needed —
everything runs from a single database file next to the executable.

## Tech stack
- C# / .NET 8 (Windows Forms)
- SQLite via `Microsoft.Data.Sqlite`

## Project layout
```
POS_System.sln
POS_System/
  POS_System.csproj
  Program.cs                 # entry point, creates PetShopPOS.db on first run
  Data/
    DatabaseHelper.cs         # schema creation, seed data, CRUD helpers
  Helpers/
    UiTheme.cs                 # color palette, fonts, styled controls
  Resources/
    sidebar_background.png     # dashboard sidebar artwork
    placeholder_product.png    # shown for products without a photo
    icon_*.png                  # sidebar (white) + card (sage) icon set
  Forms/
    FrmLogin.cs                 # login screen
    FrmDashboard.cs            # sidebar nav + summary stat cards + module cards
    FrmCategory.cs             # category CRUD
    FrmProduct.cs              # product CRUD + image upload/preview/thumbnail
    FrmCustomer.cs             # customer CRUD
    FrmBilling.cs               # image-based product tiles + shopping cart
    FrmSalesHistory.cs          # sales list + read-only sale detail popup
  ProductImages/                # uploaded product photos are copied here (runtime)
```

## How to run

### Visual Studio 2022
1. Open `POS_System.sln`.
2. Let NuGet restore `Microsoft.Data.Sqlite` (needs internet the first time only).
3. Press **F5**. `PetShopPOS.db` is created automatically next to the
   executable on first run, pre-loaded with a demo login and realistic
   pet-shop categories/products.

### .NET CLI
```bash
cd POS_System
dotnet restore
dotnet run
```

Demo login (shown as a hint on the screen, and pre-filled):
**Username:** `admin`  **Password:** `admin123`

## Database
`PetShopPOS.db` is a plain SQLite file — open it with any SQLite browser
(e.g. "DB Browser for SQLite") to inspect it directly. Schema:

| Table       | Fields                                                                 |
|-------------|--------------------------------------------------------------------------|
| Users       | UserID, Username (unique), Password, FullName                            |
| Categories  | CategoryID, CategoryName, Description                                    |
| Products    | ProductID, ProductName, CategoryID (FK), UnitPrice, StockQuantity, ImagePath |
| Customers   | CustomerID, CustomerName, ContactNumber, Address                         |
| Sales       | SaleID, SaleDate, CustomerID (FK), TotalAmount                           |
| SaleItems   | SaleItemID, SaleID (FK), ProductID (FK), Quantity, UnitPrice, LineTotal  |

Login checks the username/password directly against the `Users` table
(simple plain-text match — no hashing/encryption, kept intentionally
simple for a class assignment). Passwords are stored as-is in the
`Password` column.

## Seed data
On first run (empty database), 10 pet-supply categories are created with
realistic products and prices: **Pet Food, Toys, Collars & Walking
Accessories, Beds & Comfort Items, Grooming & Hygiene, Feeding
Accessories, Bird Accessories, Fish & Aquarium Supplies, Small Pet
Accessories, Cleaning & Waste Products.**

## Product images
- In **Manage Products**, use **Choose Image** to upload a photo (jpg/png/bmp).
  The file is copied into `ProductImages/` next to the executable and the
  file name is stored in `Products.ImagePath`.
- Products list shows a thumbnail column; products without a photo show a
  placeholder icon instead of a blank cell.
- The **Billing** screen shows every product as an image card (photo,
  name, price, stock, "Add to Cart").

## UI & design
`Helpers/UiTheme.cs` centralizes the color palette (cream background,
olive/sage accents) and a `StyleGrid()` helper so every `DataGridView`
looks consistent. The sidebar and login screen use a small paw-print
logo; the artwork in `Resources/` is simple, programmatically generated
imagery (gradients + icons), not stock photos.

## Feature notes
- **Validation**: product name/category/price/stock, category name,
  customer name, and cart quantity are all validated (name required,
  category required, price > 0, stock ≥ 0, quantity > 0 and ≤ available
  stock).
- **Billing**: adding an item checks current stock minus what's already
  in the cart; completing a sale re-validates stock, then inserts the
  `Sales` header + `SaleItems` rows and decrements `Products.StockQuantity`
  inside a single database transaction (all-or-nothing).
- **Delete protection**: categories in use by a product, products used in
  a past sale, and customers with past sales can't be deleted.
- **Dashboard stats** (Total Products, Total Categories, Total Customers,
  Today's Sales) refresh automatically every time you return from a
  module screen.
- **Exit**: confirmation prompt, then a thank-you message before closing.

## What's not included
Online payments, barcode scanning, networking, cloud/server databases,
password encryption, and multi-user concurrency — kept out intentionally
to match a standard undergraduate assignment scope.
