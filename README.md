# E-Cashier Backend - ASP.NET Core Web API

## Prerequisites
- .NET 8 SDK
- SQL Server 2022 (or SQL Server Management Studio 2022)
- Visual Studio 2022 or later

## Setup Instructions

### 1. Restore NuGet Packages
```bash
dotnet restore
```

### 2. Update Connection String
Edit `appsettings.json` and update the connection string to match your SQL Server instance:
```json
"ConnectionStrings": {
  "DefaultConnection": "Server=YOUR_SERVER_NAME;Database=ECashierDb;Trusted_Connection=True;TrustServerCertificate=True;MultipleActiveResultSets=true"
}
```

### 3. Update PayMongo API Keys
Replace the placeholder values in `appsettings.json`:
```json
"PayMongo": {
  "SecretKey": "sk_test_YOUR_ACTUAL_SECRET_KEY",
  "PublicKey": "pk_test_YOUR_ACTUAL_PUBLIC_KEY"
}
```

### 4. Run EF Core Migrations

#### Using Package Manager Console (Visual Studio):
```powershell
Add-Migration InitialCreate
Update-Database
```

#### Using .NET CLI:
```bash
dotnet ef migrations add InitialCreate
dotnet ef database update
```

### 5. Run the Application
```bash
dotnet run
```

The API will be available at:
- HTTPS: `https://localhost:7263`
- HTTP: `http://localhost:5263`
- Swagger UI: `https://localhost:7263/swagger`

## Database Tables Created

After running migrations, the following tables will be created in SQL Server:

### Students Table
- Id (int, PK, Identity)
- StudentNumber (nvarchar(50), Unique, Required)
- FullName (nvarchar(200), Required)
- Email (nvarchar(100), Unique, Required)
- PasswordHash (nvarchar(max), Required)
- Role (nvarchar(20), Default: "Student")
- CreatedAt (datetime2)

### Payments Table
- Id (int, PK, Identity)
- StudentId (int, FK to Students)
- PaymentType (nvarchar(50), Required)
- Amount (decimal(18,2), Required)
- Status (nvarchar(20), Default: "Pending")
- CreatedAt (datetime2)
- PaidAt (datetime2, Nullable)
- ExternalReferenceId (nvarchar(200), Nullable)
- Description (nvarchar(500), Nullable)

## API Endpoints

### Authentication
- `POST /api/auth/register` - Register new student
- `POST /api/auth/login` - Login and get JWT token

### Payments (Requires JWT)
- `GET /api/payments/history` - Get payment history for logged-in student
- `POST /api/payments/initiate` - Initiate new payment
- `POST /api/payments/webhook` - PayMongo webhook endpoint (no auth)
- `GET /api/payments/all` - Get all payments (Admin only)

## Testing with Swagger

1. Navigate to `https://localhost:7263/swagger`
2. Register a new student using `/api/auth/register`
3. Login using `/api/auth/login` to get JWT token
4. Click "Authorize" button and enter: `Bearer YOUR_JWT_TOKEN`
5. Test protected endpoints

## PayMongo Integration

The system uses PayMongo's Sources API to create payment checkout sessions:
- Sandbox API: `https://api.paymongo.com/v1/sources`
- Webhook events: `source.chargeable`, `payment.paid`, `payment.failed`

Configure webhook URL in PayMongo dashboard:
```
https://your-domain.com/api/payments/webhook
```
