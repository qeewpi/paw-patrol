# External APIs

This section documents the third-party services integrated into the Paw Patrol platform, including authentication requirements, data flows, rate limits, error handling, and fallback strategies.

## Google Maps Platform

**Services Used:**
1. **Maps JavaScript API** - Interactive map display with markers
2. **Geocoding API** - Address ↔ coordinates conversion
3. **Places API** - Address autocomplete (optional, not MVP)

**Authentication:**
- API Key stored in environment variables
- Frontend: `VITE_GOOGLE_MAPS_API_KEY` (restricted to specific domains)
- Backend: `GOOGLE_MAPS_API_KEY` (restricted to specific IP addresses)

**Usage:**

### Maps JavaScript API (Frontend)

**Purpose:** Display interactive map with colony markers

**Integration:**
```typescript
// Component usage - Using @vis.gl/react-google-maps (modern recommended library)
import { APIProvider, Map, AdvancedMarker } from '@vis.gl/react-google-maps';

const MapView = () => {
  const position = { lat: 14.5995, lng: 120.9842 }; // Manila default

  return (
    <APIProvider apiKey={import.meta.env.VITE_GOOGLE_MAPS_API_KEY}>
      <Map
        defaultCenter={position}
        defaultZoom={12}
        mapId={import.meta.env.VITE_GOOGLE_MAPS_MAP_ID} // Required for AdvancedMarker
        style={{ width: '100%', height: '100vh' }}
      >
        {colonies.map(colony => (
          <AdvancedMarker
            key={colony.id}
            position={{ lat: colony.latitude, lng: colony.longitude }}
            onClick={() => navigate(`/colonies/${colony.id}`)}
          />
        ))}
      </Map>
    </APIProvider>
  );
};
```

**Rate Limits:**
- **Free Tier:** $200/month credit (~28,000 map loads)
- **Expected Usage:** ~50 map loads/day × 30 days = 1,500 loads/month << 28,000
- **Cost Projection:** $0/month (well within free tier)

**Error Handling:**
- API key invalid → Show error message: "Map failed to load. Please contact support."
- Network failure → Retry up to 3 times with exponential backoff
- Quota exceeded → Fallback to static map or disable map temporarily

---

### Geocoding API (Backend)

**Purpose:** Convert addresses to coordinates when adding colonies

**Integration:**
```java
@Service
@RequiredArgsConstructor
public class GeocodingService {
    
    @Value("${google.maps.api.key}")
    private String apiKey;
    
    private final RestTemplate restTemplate;
    
    public Coordinates geocodeAddress(String address) {
        String url = String.format(
            "https://maps.googleapis.com/maps/api/geocode/json?address=%s&key=%s",
            URLEncoder.encode(address, StandardCharsets.UTF_8),
            apiKey
        );
        
        GeocodingResponse response = restTemplate.getForObject(url, GeocodingResponse.class);
        
        if (response == null || !"OK".equals(response.getStatus())) {
            throw new GeocodingException("Failed to geocode address: " + address);
        }
        
        Location location = response.getResults().get(0).getGeometry().getLocation();
        return new Coordinates(location.getLat(), location.getLng());
    }
    
    public String reverseGeocode(BigDecimal latitude, BigDecimal longitude) {
        String url = String.format(
            "https://maps.googleapis.com/maps/api/geocode/json?latlng=%s,%s&key=%s",
            latitude, longitude, apiKey
        );
        
        GeocodingResponse response = restTemplate.getForObject(url, GeocodingResponse.class);
        
        if (response == null || response.getResults().isEmpty()) {
            return String.format("%.6f, %.6f", latitude, longitude); // Fallback to coordinates
        }
        
        return response.getResults().get(0).getFormattedAddress();
    }
}
```

**Rate Limits:**
- **Free Tier:** $200/month credit (~40,000 geocoding requests)
- **Expected Usage:** ~5 new colonies/day × 30 days = 150 requests/month << 40,000
- **Cost Projection:** $0/month

**Error Handling:**
- `ZERO_RESULTS` → Return coordinates as-is, let user enter address manually
- `OVER_QUERY_LIMIT` → Cache recent results, retry after 1 second
- `REQUEST_DENIED` → Log error, throw exception (indicates configuration issue)
- Network timeout → Retry up to 3 times with exponential backoff (1s, 2s, 4s)

**Caching Strategy:**
- Cache geocoding results in memory (Google TOS allows caching for 30 days)
- Key: address string (normalized: lowercase, trimmed)
- TTL: 30 days
- Reduces API calls for duplicate addresses

**PRD Reference:** FR8, FR11, FR21

---

## API Key Management

**Security Best Practices:**

1. **Environment Variables:**
   - Never commit API keys to git
   - Store in `.env` files (gitignored)
   - Use Fly.io secrets for production: `flyctl secrets set GOOGLE_MAPS_API_KEY=xxx`

2. **API Key Restrictions (Google Cloud Console):**
   - **Frontend Key:** Restrict to HTTP referrers (e.g., `https://paw-patrol.fly.dev/*`)
   - **Backend Key:** Restrict to IP addresses (Fly.io machine IPs)
   - Enable only required APIs (Maps JavaScript API, Geocoding API)

3. **Rate Limiting:**
   - Set custom quota limits in Google Cloud Console (e.g., 100 requests/day)
   - Alert if usage exceeds 50% of quota

**Example `.env` Files:**

```bash
# Frontend (.env)
VITE_GOOGLE_MAPS_API_KEY=AIzaSyXXXXXXXXXXXXXXXXXXXXXXXXXXXX

# Backend (application.properties)
google.maps.api.key=AIzaSyYYYYYYYYYYYYYYYYYYYYYYYYYYYYYY
```

---

## Cost Monitoring

**Monthly Budget:** $0 (free tier only)

**Usage Tracking:**
- Google Cloud Console → APIs & Services → Quotas
- Set up billing alerts (alert at $10, $50, $100)
- Monitor daily API call counts

**Free Tier Limits (as of 2025):**
- Maps JavaScript API: $200/month credit
- Geocoding API: $200/month credit
- Total: $400/month credit for all Google Maps Platform services

**Expected vs. Limit:**
| Service | Expected Usage | Free Limit | % of Limit |
|---------|---------------|-----------|-----------|
| Maps JavaScript API | 1,500 loads/month | ~28,000 loads | 5% |
| Geocoding API | 150 requests/month | ~40,000 requests | 0.4% |

**Overage Strategy:**
- If approaching limits: Implement aggressive caching
- If exceeds limits: Temporarily disable map features, fallback to coordinate-only entry
- Long-term: Migrate to self-hosted OpenStreetMap (Leaflet.js) if sustained high usage

---
