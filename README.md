# NestWorks - AI-Powered E-commerce Platform

A modern React.js frontend application for a home improvement and hardware e-commerce platform with AI-powered product recommendations.

## Features

- **Modern UI/UX**: Clean, responsive design using Bootstrap 5 and custom CSS
- **AI-Powered Recommendations**: Smart product suggestions based on user preferences
- **Budget Planner**: AI-driven budget planning tool for home improvement projects
- **Multi-Role Dashboard**: Separate interfaces for customers, suppliers, and admins
- **Product Management**: Comprehensive product listing, details, and search
- **Order Management**: Complete order history and tracking system
- **Responsive Design**: Mobile-first approach with Bootstrap grid system

## Pages & Components

### Authentication
- **Login Page**: Email/password authentication with validation and JWT handling
- **Sign Up Page**: Secure user registration with bcrypt-backed password hashing (via API)

### Customer Features
- **Home/Dashboard**: Product categories, AI recommendations, and quick stats
- **Product Listing**: Filterable product grid with search and sorting
- **Product Detail**: Detailed product view with specifications and reviews
- **Budget Planner**: AI-powered budget planning with product recommendations
- **Shopping Cart**: Cart management with promo codes and checkout
- **Order History**: Order tracking and management

### Business Features
- **Supplier Dashboard**: Product management, sales analytics, and order processing
- **Admin Dashboard**: Platform analytics, user management, and system monitoring

## Technology Stack

- **Frontend**: React.js 18
- **Routing**: React Router DOM 6
- **Styling**: Bootstrap 5 + Custom CSS
- **Icons**: Font Awesome 6
- **Backend**: Node.js + Express + MongoDB (secure authentication)
- **Data**: Static JSON files (catalog demo data) + MongoDB collections (accounts)

## Getting Started

### Prerequisites

- Node.js (version 14 or higher)
- npm or yarn package manager

### Installation

Clone the repository (or open in Cursor) and install dependencies for both frontend and backend:

```bash
# Frontend
cd NestWork
npm install

# Backend
cd ../server
npm install
```

### Running the Stack Locally

1. **Configure environment variables**  
   Create `server/.env` with:
   ```
   PORT=5000
   JWT_SECRET=replace-with-strong-secret
   MONGO_URI=mongodb://127.0.0.1:27017/nestworks
   MONGO_DB_NAME=nestworks
   ```

2. **Start MongoDB** (local instance or point `MONGO_URI` to Atlas/remote)

3. **Run the backend**  
   ```bash
   npm run dev --prefix server
   ```

4. **Run the frontend**  
   ```bash
   npm start --prefix NestWork
   ```

5. Open `http://localhost:3000`. The CRA dev server proxies API calls to `http://localhost:5000`.

### Available Scripts

- `npm start` - Runs the app in development mode
- `npm build` - Builds the app for production
- `npm test` - Launches the test runner
- `npm eject` - Ejects from Create React App (one-way operation)
- `npm run dev --prefix server` - Starts the Express backend with nodemon

## Authentication Flow

The frontend is fully wired to the Express API located in `server/`:

| Method | Route                 | Description                                             |
|--------|-----------------------|---------------------------------------------------------|
| POST   | `/api/auth/register`  | Creates a new user, hashes password with bcrypt, returns JWT |
| POST   | `/api/auth/login`     | Authenticates credentials, returns JWT + profile        |
| GET    | `/api/auth/me`        | Fetches current user profile (requires `Authorization` header) |
| GET    | `/api/auth/user-only` | Sample protected route for users/admins                 |
| GET    | `/api/auth/admin-only`| Sample protected route requiring `admin` role           |

JWT tokens are stored in `localStorage` and automatically cleared on logout. Protected React routes use a custom `ProtectedRoute` component to enforce authentication and role checks.

## Project Structure

```
src/
├── components/          # Reusable UI components
│   ├── Navbar.js       # Navigation bar
│   ├── Footer.js       # Footer component
│   └── Sidebar.js      # Sidebar navigation
├── pages/              # Page components
│   ├── Login.js        # Login page (API-integrated)
│   ├── Signup.js       # Registration page (API-integrated)
│   ├── Dashboard.js    # Authenticated account hub
│   ├── Home.js         # Home dashboard
│   ├── ProductListing.js # Product catalog
│   ├── ProductDetail.js  # Product details
│   ├── BudgetPlanner.js  # AI budget planner
│   ├── Cart.js         # Shopping cart
│   ├── OrderHistory.js # Order management
│   ├── SupplierDashboard.js # Supplier interface
│   └── AdminDashboard.js    # Admin interface
├── data/               # Static data files
│   ├── products.json   # Product catalog
│   ├── users.json      # User data
│   └── orders.json     # Order history
├── context/
│   └── AuthContext.js  # Auth state management + storage helpers
├── components/
│   └── ProtectedRoute.js # Wrapper for protected React Router segments
├── App.js              # Main app component
├── App.css             # Custom styles
├── index.js            # App entry point
└── index.css           # Global styles
```

## Key Features

### AI-Powered Recommendations
- Smart product suggestions based on budget and preferences
- Category-based filtering and sorting
- Personalized home improvement packages

### Budget Planning
- Interactive budget calculator
- AI-driven product recommendations within budget
- Visual budget usage indicators

### Multi-Role Interface
- **Customer**: Browse, search, and purchase products
- **Supplier**: Manage products, view sales analytics, process orders
- **Admin**: Monitor platform performance, manage users and orders

### Responsive Design
- Mobile-first approach
- Bootstrap 5 grid system
- Custom CSS for enhanced styling
- Font Awesome icons throughout

## Demo Data

The application includes comprehensive demo data:
- 8 sample products across different categories
- User accounts for different roles
- Order history and analytics data
- Supplier and admin dashboard data

## Customization

### Styling
- Modify `src/App.css` for custom styles
- Bootstrap 5 classes are used throughout
- CSS custom properties for consistent theming

### Data
- Update JSON files in `src/data/` to modify demo data
- Add new products, users, or orders as needed

### Components
- All components are modular and reusable
- Easy to extend with new features
- Consistent prop patterns across components

## Browser Support

- Chrome (latest)
- Firefox (latest)
- Safari (latest)
- Edge (latest)

## License

This project is created for demonstration purposes. Feel free to use and modify as needed.

## Support

For questions or issues, please refer to the code comments or create an issue in the project repository.
