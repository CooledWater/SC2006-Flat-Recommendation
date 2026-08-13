# FlatMatch - HDB Resale Finder & Affordability Tool

A Next.js web application that helps users find and evaluate HDB resale flats in Singapore based on proximity to amenities, affordability, and personal preferences.

## Features

### Smart Property Search
- Browse thousands of HDB resale listings with infinite scroll
- Filter by neighbourhood, price range, and room type
- Autocomplete search for neighbourhoods
- Bookmark favorite listings

### Affordability Scoring (1-10)
- Real-time affordability calculation based on:
  - Down payment budget vs required down payment
  - Monthly income vs mortgage repayment (30% MSR rule)
  - Age and remaining lease constraints
- Uses HDB financing rules:
  - 75% LTV (Loan-to-Value)
  - 3.1% annual interest rate
  - Tenure: min(25 years, 65-age, remaining lease-20)

### HDBFinder - Multi-Criteria Scoring
- Score flats based on weighted preferences:
  - Distance to MRT stations
  - Distance to schools
  - Distance to hospitals
  - Affordability (based on user profile)
- Select up to 3 towns and preferred flat type
- Finds the cheapest recent listing (last 24 months) per town

### Interactive Maps
- Property location visualization using Leaflet
- View nearby amenities:
  - MRT stations
  - Schools
  - Hospitals
- Distance calculations and route visualization

### User Profiles
- Save personal financial information:
  - Annual income
  - Age
  - Down payment budget
  - Citizenship status
  - Preferred flat type and area
- Persistent affordability scores across listings

## Tech Stack

- **Framework**: Next.js 15.5.6 (App Router)
- **Language**: TypeScript
- **Styling**: Tailwind CSS
- **Database**: MongoDB (Mongoose ODM)
- **Maps**: Leaflet + React-Leaflet
- **APIs**: 
  - OneMap API (Singapore geocoding & amenities)
  - Custom HDB data API

## Getting Started

### Prerequisites

- Node.js 18+ and npm
- MongoDB database
- OneMap API credentials (register at [OneMap](https://www.onemap.gov.sg/apidocs/))

### Installation

1. Clone the repository:
   ```bash
   git clone https://github.com/YAROUDIO/2006-Project-HDB-Finder.git
   cd hdb-app
   ```

2. Install dependencies:
   ```bash
   npm install
   ```

3. Create `.env.local` file in the project root:
   ```env
   # MongoDB
   MONGODB_URI=mongodb+srv://your-connection-string

   # OneMap API (Singapore mapping service)
   ONEMAP_EMAIL=your-onemap-email
   ONEMAP_PASSWORD=your-onemap-password

   # Optional: Base URL for API calls
   NEXT_PUBLIC_BASE_URL=http://localhost:3000
   ```

4. Run the development server:
   ```bash
   npm run dev
   ```

5. Open [http://localhost:3000](http://localhost:3000) in your browser

### Building for Production

```bash
npm run build
npm start
```

## Project Structure

```
hdb-app/
├── src/app/                    # Next.js App Router pages
│   ├── (auth)/                # Authentication pages
│   │   ├── login/
│   │   └── register/
│   ├── (display)/             # Main app pages
│   │   ├── finder/           # HDBFinder scoring tool
│   │   ├── listing/          # Browse all listings
│   │   │   └── [id]/         # Individual listing detail + map
│   │   └── bookmarks/        # Saved listings
│   ├── home/                 # Homepage with featured listings
│   ├── account/              # User account overview
│   ├── userinfo/             # Edit user profile
│   ├── api/                  # API routes
│   │   ├── hdbdata/         # HDB listings CRUD
│   │   ├── finder/          # Multi-criteria scoring
│   │   ├── score-batch/     # Batch affordability scoring
│   │   ├── onemap/          # OneMap integration
│   │   ├── coords/          # Geocoding
│   │   └── bookmarks/       # Bookmark management
│   └── globals.css          # Global styles
├── lib/                      # Utility libraries
│   ├── affordability.ts     # Affordability calculation logic
│   ├── mongoose.ts          # MongoDB connection
│   └── resale.ts            # HDB data helpers
├── models/                   # Mongoose schemas
│   └── User.ts              # User model
├── components/              # React components
│   └── AffordabilityWidget.tsx
└── public/                  # Static assets
```

## Key Features Explained

### Affordability Calculation

The app uses a sophisticated two-factor scoring system:

**1. Down Payment Score (1-10):**
- Compares user's budget vs required down payment (25% of lower of price/valuation)
- Asymmetric scoring: harsher penalties below 100%, generous rewards above
- 200%+ buffer = 10/10, exactly 100% = 5/10, <70% = 1/10

**2. Monthly Payment Score (1-10):**
- Follows HDB's 30% MSR (Mortgage Servicing Ratio) rule
- Monthly payment ≤ 30% of monthly income = 10/10
- Every 5% above threshold drops score by 2 points

**Final Score:** Minimum of both scores (limited by worst constraint)

### Finder Scoring Algorithm

Combines multiple weighted criteria:
- **MRT Distance**: Closer is better
- **School Distance**: Closer is better
- **Hospital Distance**: Closer is better
- **Affordability**: Based on user's financial profile

Each criterion scored 0-100, then weighted by user preferences and combined into a final composite score.

## Database Schema

### User Model
```typescript
{
  username: String (unique, required)
  email: String (optional)
  password: String (hashed, required)
  income: String
  age: Number
  citizenship: String
  flatType: String
  downPaymentBudget: Number
  area: String
}
```

### Bookmark Model
```typescript
{
  username: String
  block: String
  street_name: String
  flat_type: String
  month: String
  resale_price: String
  compositeKey: String (unique identifier)
}
```

## API Endpoints

### Public Routes
- `GET /api/hdbdata` - Fetch HDB listings with pagination and filters
- `GET /api/coords` - Geocode address to coordinates
- `GET /api/onemap/search` - OneMap address search
- `GET /api/onemap/nearby` - Find nearby amenities

### Protected Routes (require login)
- `GET /api/userinfo` - Get user profile
- `POST /api/userinfo` - Update user profile
- `GET /api/bookmarks` - Get user's bookmarks
- `POST /api/bookmarks` - Add bookmark
- `DELETE /api/bookmarks` - Remove bookmark
- `POST /api/finder` - Run multi-criteria scoring
- `POST /api/score-batch` - Batch affordability scoring

## Known Issues & Limitations

1. **Price Range Required**: Listing page filters require a price range to work properly
2. **OneMap Credentials**: Map features require valid OneMap API credentials
3. **Data Source**: Uses static HDB resale data (may not reflect latest transactions)
4. **Affordability**: Assumes CPF usage and standard HDB loan terms

## Future Enhancements

- [ ] Real-time HDB data integration
- [ ] Advanced filtering (floor level, flat model, storey range)
- [ ] Comparison tool for multiple listings
- [ ] Email notifications for new listings
- [ ] Mobile app version
- [ ] Transit time calculations
- [ ] Neighborhood insights and statistics

## Contributing

This is a school project. Contributions are welcome through pull requests.

## License

This project is for educational purposes.

## Authors

- SC2006 Group Project

## Acknowledgments

- Data.gov.sg for HDB resale price data
- OneMap for Singapore mapping services
- Next.js team for the excellent framework
