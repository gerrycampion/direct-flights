# Direct Flights Finder

A web application to discover all direct flight destinations from any airport, complete with pricing information and currency exchange rates.

## Features

- **Airport Search**: Quick search by 3-letter IATA code (e.g., JFK, LHR, CDG)
- **Comprehensive Direct Flight Listings**: Fetches up to 300 scheduled flights to maximize destination coverage
- **Intelligent Data Enrichment**: Automatically looks up destination details to provide accurate city, country, and currency information
- **Dynamic Country Data**: Uses REST Countries API for always up-to-date country names, codes, and currencies (250+ countries)
- **Full Country Names**: Displays complete country names (e.g., "United States", "United Kingdom") instead of abbreviations for better readability
- **Smart City/Country Parsing**: Extracts proper city and region names from timezone data (e.g., "America/New_York" → "New York" in "America")
- **Sortable Table**: Click any column header to sort results (airport name, city, country, location, airport code, price, exchange rate, or trends)
- **Domestic/International Filter**: Location column clearly indicates whether each destination is a domestic or international flight
- **Clear Column Structure**: Separate columns for airport name, city (with TripAdvisor link), country, location type, and airport code for easy filtering and navigation
- **Destination Links**: Clickable city names linking to TripAdvisor for destination research
- **Google Flights Integration**: Airport codes link directly to Google Flights search
- **Price Range**: Estimated flight costs based on distance
- **Currency Conversion**: Real-time exchange rates between departure and destination currencies
- **Historical Rate Data**: Real historical exchange rates (not simulated) from Frankfurter API
- **Rate Trends**: Visual indicators showing actual exchange rate changes over the past month and year, each linked to corresponding currency chart views
- **Smart Caching**: Reduces API calls by caching results for 24 hours
- **Responsive Design**: Works on desktop, tablet, and mobile devices

## Getting Started

### Quick Start (No API Keys Needed)

The app works out of the box with fallback data. Simply open `index.html` in your web browser:

```bash
# Windows
start index.html

# Mac
open index.html

# Linux
xdg-open index.html
```

### Using APIs for Real-Time Data (Optional)

The app works great out of the box with fallback data. For real-time flight information:

#### 1. API Ninjas (Airport Search)

- Sign up at: https://api-ninjas.com/
- Free tier: **50,000 requests/month**
- Lookup any airport by 3-letter IATA code

#### 2. AviationStack (Flight Data)

- Sign up at: https://aviationstack.com/
- Free tier: **100 requests/month**
- Real-time scheduled flight data for finding direct routes

#### Setup Instructions

**Option 1: Using config.js (Recommended - Keeps keys secure)**

1. Copy `config.example.js` to `config.js`:

   ```bash
   # Windows
   copy config.example.js config.js

   # Mac/Linux
   cp config.example.js config.js
   ```

2. Open `config.js` in a text editor

3. Add your API keys:

   ```javascript
   window.API_CONFIG = {
     apininjas: "your_api_ninjas_key_here",
     aviationstack: "your_aviationstack_key_here",
   };
   ```

4. Save and open `index.html` in your browser

**Option 2: Edit index.html directly (Quick but less secure)**

1. Open `index.html` in a text editor
2. Find the `API_KEYS` configuration section (around line 425)
3. Add your API keys directly in the code
4. Save and open in your browser

**Note**:

- `config.js` is git-ignored, keeping your API keys private
- The app works without API keys using fallback data for major airports
- With API keys: Real-time data for any airport (API limits apply)

## How It Works

### With API Keys Configured

1. **Airport Search**:
   - Enter 3-letter IATA codes (e.g., "JFK", "LHR", "CDG")
   - API Ninjas looks up airport details
   - Fast, accurate results for 28,000+ airports
2. **Flight Routes**:
   - AviationStack fetches real-time scheduled flights (paginated up to 300 flights)
   - Automatically looks up destination airport details via API Ninjas when available
   - Intelligently parses city names from airport names and timezone data
   - Extracts region/country from timezone strings (e.g., "America/New_York" → City: "New York", Country: "America")
   - Falls back to curated data if API unavailable
3. **Currency Rates**: Gets live exchange rates from exchangerate-api.com (no key needed)
4. **Caching**:
   - Airport search results cached per code for 24 hours
   - Flight route data cached per airport for 24 hours (critical for 100/month limit!)
   - Currency rates cached per pair for 24 hours
5. **Pricing**: Estimates flight costs based on distance

### Without API Keys

- Uses fallback data for 15 major airports (JFK, LHR, CDG, etc.)
- Curated route data for these airports
- Still fetches real-time currency exchange rates
- All features work normally

## Architecture

### Current APIs Integrated

1. **API Ninjas - Airports** (Optional)
   - **Purpose**: Airport lookup by IATA code + destination airport details enrichment
   - **Endpoint Used**: `/airports?iata=CODE`
   - **Free Tier**: 50,000 requests/month
   - **Optimization**: Results cached for 24 hours
   - **Fallback**: 15 major airports hardcoded
   - **Usage**: Primary airport search + automatic destination lookup for complete data

2. **AviationStack - Flights** (Optional)
   - **Purpose**: Real-time scheduled flight data for finding direct routes
   - **Endpoint Used**: `/flights?dep_iata=CODE&flight_status=scheduled&offset=X`
   - **Pagination**: Fetches up to 3 pages (300 flights) to maximize destination coverage
   - **Free Tier**: 100 requests/month (⚠️ pagination uses 3 requests per search)
   - **Critical**: Caching prevents exceeding limits - each airport search is cached for 24 hours
   - **Fallback**: Curated route data for major airports
   - **Enhancement**: Automatically enriches destination data with API Ninjas lookups

3. **Frankfurter API** (No key required)
   - **Purpose**: Real-time and historical currency conversion
   - **Endpoints Used**:
     - `/latest?from=X&to=Y` - Current rates
     - `/YYYY-MM-DD?from=X&to=Y` - Historical rates
   - **Free Tier**: Unlimited requests
   - **Always Active**: Works without configuration
   - **Data Quality**: Real historical data (not simulated)

4. **REST Countries API** (No key required)
   - **Purpose**: Dynamic country data (names, codes, currencies)
   - **Endpoint Used**: `https://restcountries.com/v3.1/all?fields=cca2,name,altSpellings,currencies`
   - **Free Tier**: Unlimited requests
   - **Caching**: 30 days (country data rarely changes)
   - **Coverage**: 250+ countries with official names, currencies, and ISO codes
   - **Benefit**: Always up-to-date, no manual maintenance required
   - **Fallback**: Minimal hardcoded data for ~40 major countries if API fails

### Caching System

The app uses LocalStorage with intelligent cache durations:

**What's Cached:**

- Airport lookup results (per IATA code) - 24 hours
- Flight route data (per departure airport) - 24 hours
- Currency exchange rates (per currency pair) - 24 hours
- Country data (names, codes, currencies) - 30 days
- Flight route data (per departure airport)
- Currency exchange rates (per currency pair)

**Benefits:**

- Reduces API usage by ~90% after initial searches
- Instant results for repeated searches
- Easily stays within free tier limits (100-50,000 requests/month)
- Works offline for previously searched airports

### Data Flow

```
User Enters Code → Check Cache → API Lookup → Display & Cache
                       ↓              (if needed)
                  Return Cached
```

## Upgrading to Premium APIs

For production use with higher volume or more features, consider:

### Flight Pricing APIs

- **Amadeus Flight Offers** - Real-time pricing (paid)
- **Skyscanner** - Comprehensive flight search (paid)
- **Kiwi.com Tequila** - Multi-city routes (tiered pricing)

**Note**: The app now uses Frankfurter API which provides real historical exchange rate data for free, so premium currency APIs are generally not needed unless you require multi-year historical data or specific currency pairs not supported by Frankfurter.

## Technology Stack

- **HTML5**: Structure and semantic markup
- **CSS3**: Modern styling with gradients, flexbox, and animations
- **JavaScript (ES6+)**: Client-side functionality with async/await, table sorting
- **LocalStorage**: Client-side caching system
- **Free APIs Integrated**:
  - **API Ninjas** - Airport data lookup (optional, 50k/month)
  - **AviationStack** - Real-time scheduled flights (optional, 100/month)
  - **Frankfurter API** - Real-time and historical currency data (no key needed, unlimited)
- **Fallback Data**:
  - Curated direct flight routes for 15 major airports (JFK, LHR, CDG, DXB, etc.)
  - Ensures app works without API keys or when limits are reached
- **Interactive Features**:
  - Sortable table columns (click any header to sort)
  - Visual sort indicators with ascending/descending states
- **External Links**:
  - TripAdvisor - Destination search
  - Google Flights - Flight booking
  - XE.com - Currency charts

## API Rate Limits & Caching

**Free Tier Limits:**

- Frankfurter API: Unlimited requests (no key required)
- API Ninjas: 50,000 requests/month (optional) - Used for airport search + destination enrichment
- AviationStack: 100 requests/month (⚠️ **very limited**)
  - **Important**: Pagination uses 3 API calls per airport search (to fetch 300 flights)
  - This means ~33 unique airports can be searched per month
  - Caching is absolutely critical - each airport is cached for 24 hours

**Built-in Caching:**

The app automatically caches API responses in LocalStorage for 24 hours, which:

- Reduces actual API calls by ~90%
- **CRITICAL for AviationStack's 100/month limit**
- Makes the free tiers sufficient for most personal use
- Provides instant results for repeated searches

**Note on Flight Routes:**

The app uses AviationStack's real-time scheduled flights API with intelligent pagination (up to 3 pages / 300 flights) to discover comprehensive direct flight destinations. It automatically enriches destination data by looking up airport details via API Ninjas, ensuring accurate city, country, and currency information. Given the tight 100 requests/month limit (3 calls per airport due to pagination), caching is absolutely essential. The app also includes fallback data for 15 major airports to ensure functionality even without an API key or when the limit is reached.

## Future Enhancements

- [ ] Add user authentication to save favorite airports and routes
- [x] ~~Implement caching for API responses~~ ✅ Completed
- [ ] Add filters (price range, distance, airline preferences)
- [ ] Include flight duration and distance information
- [ ] Add seasonal price trends and best time to book
- [ ] Integrate weather data for destinations
- [ ] Add map visualization of routes
- [ ] Include visa requirement information
- [ ] Add real-time flight pricing (requires paid API)
- [ ] Implement server-side API proxy to hide API keys
- [ ] Include visa requirement information
- [ ] Add booking integration
- [ ] Implement server-side API proxy to hide API keys

## Browser Compatibility

The application works on all modern browsers:

- Chrome/Edge 90+
- Firefox 88+
- Safari 14+
- Opera 76+

## License

This project is free to use and modify for personal and commercial purposes.

## Notes

- **Pricing**: Flight prices are estimates based on distance; for accurate pricing, click through to Google Flights
- **Exchange rates**: Updated in real-time via free API (no configuration needed)
- **Caching**: All API data is cached for 24 hours to optimize performance and stay within free tier limits
- **CORS**: Some browsers may block API calls when opened as a local file. Use a local web server if you encounter issues
- **API Keys**: The app works without API keys but provides limited airport coverage. Add free API keys for full functionality
- Always verify prices, routes, and availability on the respective booking platforms before making travel decisions

## Troubleshooting

### CORS Errors

If you see CORS errors in the browser console:

1. **Use a local web server** instead of opening the file directly:

   ```bash
   python -m http.server 8000
   # Then open http://localhost:8000
   ```

2. **Or use a browser extension** like "Allow CORS" (for development only)

3. **Or use a different browser** - some browsers are more strict than others

### API Not Working

1. **Check your API keys** - Make sure they're correctly added to the `API_KEYS` object
2. **Check API quotas** - Free tiers have monthly limits. Check your dashboard
3. **Check browser console** - Look for error messages that explain what went wrong
4. **Fallback data** - The app will automatically use fallback data if APIs fail

### Cache Issues

If you're seeing old data:

1. Open browser DevTools → Application → Local Storage
2. Clear the storage for the site
3. Refresh the page

Or update the `CACHE_DURATION` variable in the code (line ~428)
