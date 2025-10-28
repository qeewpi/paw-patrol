# Testing Strategy

## Frontend Testing

**Unit Tests (Vitest):**

```typescript
// src/components/Button.test.tsx
import { render, screen, fireEvent } from '@testing-library/react';
import { describe, it, expect, vi } from 'vitest';
import { Button } from './Button';

describe('Button', () => {
  it('renders with text', () => {
    render(<Button>Click me</Button>);
    expect(screen.getByText('Click me')).toBeInTheDocument();
  });

  it('calls onClick when clicked', () => {
    const handleClick = vi.fn();
    render(<Button onClick={handleClick}>Click</Button>);
    fireEvent.click(screen.getByText('Click'));
    expect(handleClick).toHaveBeenCalledTimes(1);
  });
});
```

**Run Frontend Tests:**

```bash
npm run test -w frontend
npm run test:coverage -w frontend
```

---

## Backend Testing

**Controller Tests (MockMvc):**

```java
@WebMvcTest(ColonyController.class)
class ColonyControllerTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @MockBean
    private ColonyService colonyService;
    
    @Test
    @WithMockUser(roles = "USER")
    void getColony_WhenExists_ReturnsColony() throws Exception {
        UUID colonyId = UUID.randomUUID();
        ColonyDTO colony = ColonyDTO.builder()
            .id(colonyId)
            .name("Test Colony")
            .build();
        
        when(colonyService.getColonyById(colonyId)).thenReturn(colony);
        
        mockMvc.perform(get("/api/colonies/{id}", colonyId))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.id").value(colonyId.toString()))
            .andExpect(jsonPath("$.name").value("Test Colony"));
    }
}
```

**Service Tests:**

```java
@SpringBootTest
@Transactional
class ColonyServiceTest {
    
    @Autowired
    private ColonyService colonyService;
    
    @Autowired
    private ColonyRepository colonyRepository;
    
    @Test
    void createColony_WhenValid_SavesColony() {
        CreateColonyRequest request = CreateColonyRequest.builder()
            .name("New Colony")
            .latitude(new BigDecimal("14.5995"))
            .longitude(new BigDecimal("120.9842"))
            .build();
        
        ColonyDTO result = colonyService.createColony(request, UUID.randomUUID());
        
        assertNotNull(result.getId());
        assertEquals("New Colony", result.getName());
    }
}
```

**Run Backend Tests:**

```bash
cd backend
mvn test
mvn verify # Includes integration tests
```

---
