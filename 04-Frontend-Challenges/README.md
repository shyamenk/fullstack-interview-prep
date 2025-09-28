# Frontend Coding Challenges

## 🎯 Challenge Categories

### Component Building Challenges
React component implementation and pattern recognition

### UI/UX Implementation Challenges
Design system creation and responsive layouts

### Performance & Optimization Challenges
Code optimization and web performance improvements

### State Management Challenges
Complex state scenarios and data flow patterns

## 🧩 Component Building Challenges

### 1. Auto-Complete Search Component
**Difficulty**: Medium
**Time Limit**: 45 minutes

**Requirements**:
- [ ] Search input with dropdown suggestions
- [ ] Debounced API calls
- [ ] Keyboard navigation (arrow keys, enter, escape)
- [ ] Loading states and error handling
- [ ] Highlight matching text in suggestions

**Technical Focus**:
- Custom hooks (useDebounce, useApi)
- Event handling and keyboard navigation
- Performance optimization
- Accessibility (ARIA attributes)

**Example Implementation**:
```jsx
function AutoComplete({ onSelect, placeholder, apiEndpoint }) {
  const [query, setQuery] = useState('');
  const [suggestions, setSuggestions] = useState([]);
  const [isLoading, setIsLoading] = useState(false);
  const [selectedIndex, setSelectedIndex] = useState(-1);

  const debouncedQuery = useDebounce(query, 300);

  // Implementation details...

  return (
    <div className="autocomplete">
      <input
        value={query}
        onChange={handleInputChange}
        onKeyDown={handleKeyDown}
        placeholder={placeholder}
        aria-expanded={suggestions.length > 0}
        aria-activedescendant={selectedIndex >= 0 ? `option-${selectedIndex}` : ''}
      />
      {/* Suggestions dropdown */}
    </div>
  );
}
```

### 2. Infinite Scroll Component
**Difficulty**: Medium
**Time Limit**: 40 minutes

**Requirements**:
- [ ] Load more items as user scrolls
- [ ] Loading indicator at bottom
- [ ] Handle end of data scenario
- [ ] Smooth scrolling experience
- [ ] Error handling and retry mechanism

**Technical Focus**:
- Intersection Observer API
- Virtual scrolling for performance
- Memory management
- User experience optimization

### 3. Modal Component with Portal
**Difficulty**: Easy-Medium
**Time Limit**: 30 minutes

**Requirements**:
- [ ] Render modal outside component tree
- [ ] Focus management and trap
- [ ] ESC key to close
- [ ] Click outside to close
- [ ] Prevent background scroll
- [ ] Animation support

### 4. Data Table with Sorting & Filtering
**Difficulty**: Hard
**Time Limit**: 60 minutes

**Requirements**:
- [ ] Column sorting (ascending/descending)
- [ ] Row filtering by multiple criteria
- [ ] Pagination controls
- [ ] Row selection (single/multiple)
- [ ] Responsive design
- [ ] Export functionality

## 🎨 UI/UX Implementation Challenges

### 5. Responsive Dashboard Layout
**Difficulty**: Medium
**Time Limit**: 50 minutes

**Requirements**:
- [ ] Grid-based layout with draggable widgets
- [ ] Responsive breakpoints (mobile, tablet, desktop)
- [ ] Dark/light theme toggle
- [ ] Sidebar navigation with collapse
- [ ] Custom CSS properties for theming

**Technical Focus**:
- CSS Grid and Flexbox mastery
- CSS custom properties
- Responsive design patterns
- Theme management

### 6. Image Gallery with Lazy Loading
**Difficulty**: Medium
**Time Limit**: 45 minutes

**Requirements**:
- [ ] Masonry/Pinterest-style layout
- [ ] Lazy load images on scroll
- [ ] Lightbox modal for full-size view
- [ ] Touch/swipe support for mobile
- [ ] Image optimization and WebP support

### 7. Form Builder Interface
**Difficulty**: Hard
**Time Limit**: 75 minutes

**Requirements**:
- [ ] Drag and drop form elements
- [ ] Dynamic form validation
- [ ] Conditional field display
- [ ] Form preview and submission
- [ ] Export form configuration as JSON

## ⚡ Performance & Optimization Challenges

### 8. Virtual Scrolling List
**Difficulty**: Hard
**Time Limit**: 60 minutes

**Requirements**:
- [ ] Render only visible items
- [ ] Handle dynamic item heights
- [ ] Smooth scrolling experience
- [ ] Support for 10,000+ items
- [ ] Search and filter functionality

**Technical Focus**:
- Performance optimization
- Memory management
- Scroll event handling
- DOM manipulation efficiency

### 9. Code Splitting & Lazy Loading
**Difficulty**: Medium
**Time Limit**: 40 minutes

**Requirements**:
- [ ] Implement route-based code splitting
- [ ] Component-level lazy loading
- [ ] Loading states for async components
- [ ] Error boundaries for failed loads
- [ ] Preloading strategies

### 10. Web Worker Integration
**Difficulty**: Hard
**Time Limit**: 55 minutes

**Requirements**:
- [ ] Offload heavy computations to Web Worker
- [ ] Progress reporting from worker
- [ ] Error handling and fallbacks
- [ ] Data transfer optimization
- [ ] Worker lifecycle management

## 🗃️ State Management Challenges

### 11. Redux-like State Manager
**Difficulty**: Hard
**Time Limit**: 70 minutes

**Requirements**:
- [ ] Create custom state management library
- [ ] Actions, reducers, and store implementation
- [ ] Middleware support
- [ ] DevTools integration
- [ ] TypeScript support

**Technical Focus**:
- State management patterns
- Observer pattern
- TypeScript generics
- Developer experience

### 12. Real-time Chat Component
**Difficulty**: Hard
**Time Limit**: 80 minutes

**Requirements**:
- [ ] WebSocket connection management
- [ ] Message history with pagination
- [ ] Typing indicators
- [ ] Online/offline status
- [ ] Message delivery status
- [ ] File upload support

### 13. Shopping Cart with Persistence
**Difficulty**: Medium
**Time Limit**: 50 minutes

**Requirements**:
- [ ] Add/remove items functionality
- [ ] Quantity management
- [ ] Price calculations with discounts
- [ ] Local storage persistence
- [ ] Sync across browser tabs
- [ ] Guest checkout flow

## 🧪 Testing Challenges

### 14. Test-Driven Component Development
**Difficulty**: Medium
**Time Limit**: 60 minutes

**Requirements**:
- [ ] Write tests first, then implement component
- [ ] Test user interactions and edge cases
- [ ] Mock external dependencies
- [ ] Achieve 100% test coverage
- [ ] Integration tests with React Testing Library

### 15. E2E Testing Scenario
**Difficulty**: Medium
**Time Limit**: 45 minutes

**Requirements**:
- [ ] Write Cypress/Playwright tests for user journey
- [ ] Test form submissions and validations
- [ ] Handle async operations and loading states
- [ ] Test responsive design breakpoints
- [ ] Performance testing integration

## 🎯 Challenge Solving Strategy

### Before Starting (5 minutes)
1. **Read requirements carefully** - Don't miss any details
2. **Ask clarifying questions** - Scope and constraints
3. **Plan your approach** - Break down into smaller tasks
4. **Choose your tools** - React patterns and libraries

### During Implementation (80% of time)
1. **Start with basic structure** - Get something working first
2. **Implement core functionality** - Focus on requirements
3. **Add polish and edge cases** - Time permitting
4. **Test manually** - Verify functionality works

### Code Review (15% of time)
1. **Check all requirements** - Make sure nothing is missed
2. **Optimize performance** - Remove unnecessary re-renders
3. **Improve code quality** - Clean up and add comments
4. **Test edge cases** - Error states and boundary conditions

## 📝 Common Patterns & Solutions

### Debouncing Hook
```jsx
function useDebounce(value, delay) {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const handler = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    return () => {
      clearTimeout(handler);
    };
  }, [value, delay]);

  return debouncedValue;
}
```

### Intersection Observer Hook
```jsx
function useIntersectionObserver(options) {
  const [isIntersecting, setIsIntersecting] = useState(false);
  const ref = useRef();

  useEffect(() => {
    const observer = new IntersectionObserver(([entry]) => {
      setIsIntersecting(entry.isIntersecting);
    }, options);

    if (ref.current) {
      observer.observe(ref.current);
    }

    return () => observer.disconnect();
  }, [options]);

  return [ref, isIntersecting];
}
```

### Portal Component
```jsx
function Portal({ children, target = document.body }) {
  const [container] = useState(() => document.createElement('div'));

  useEffect(() => {
    target.appendChild(container);
    return () => target.removeChild(container);
  }, [container, target]);

  return ReactDOM.createPortal(children, container);
}
```

## 🎪 Live Coding Tips

### Setup Efficiency
- [ ] **Use Create React App** or **Vite** for quick setup
- [ ] **Have boilerplate ready** - Common hooks and utilities
- [ ] **Know your shortcuts** - VS Code snippets and extensions
- [ ] **Prepare common imports** - React, useState, useEffect

### Code Organization
- [ ] **Start with component structure** - Props and basic JSX
- [ ] **Add state management** - useState and useEffect
- [ ] **Implement event handlers** - User interactions
- [ ] **Add styling** - CSS modules or styled-components
- [ ] **Handle edge cases** - Loading, error, empty states

### Communication During Coding
- [ ] **Think out loud** - Explain your approach
- [ ] **Ask for feedback** - "Does this approach make sense?"
- [ ] **Explain trade-offs** - "I'm choosing X over Y because..."
- [ ] **Handle suggestions** - Be open to interviewer input

## 📊 Practice Schedule

### Daily Practice (45-60 minutes)
- **Day 1**: Component building challenges (1-4)
- **Day 2**: UI/UX implementation challenges (5-7)
- **Day 3**: Performance challenges (8-10)
- **Day 4**: State management challenges (11-13)
- **Day 5**: Testing challenges (14-15)
- **Day 6**: Review and optimize previous solutions
- **Day 7**: Timed practice with random challenges

### Weekly Goals
- **Week 1**: Complete all 15 challenges at your own pace
- **Week 2**: Time yourself and aim for target completion times
- **Week 3**: Practice explaining solutions out loud
- **Week 4**: Mock live coding sessions with feedback

## ✅ Progress Tracker

### Challenge Completion
- [ ] Auto-Complete Search (Challenge 1)
- [ ] Infinite Scroll (Challenge 2)
- [ ] Modal with Portal (Challenge 3)
- [ ] Data Table (Challenge 4)
- [ ] Responsive Dashboard (Challenge 5)
- [ ] Image Gallery (Challenge 6)
- [ ] Form Builder (Challenge 7)
- [ ] Virtual Scrolling (Challenge 8)
- [ ] Code Splitting (Challenge 9)
- [ ] Web Worker Integration (Challenge 10)
- [ ] State Manager (Challenge 11)
- [ ] Real-time Chat (Challenge 12)
- [ ] Shopping Cart (Challenge 13)
- [ ] TDD Component (Challenge 14)
- [ ] E2E Testing (Challenge 15)

### Skill Assessment
- [ ] Can complete easy challenges in < 30 minutes
- [ ] Can complete medium challenges in < 45 minutes
- [ ] Can complete hard challenges in < 70 minutes
- [ ] Can explain approach clearly while coding
- [ ] Can handle unexpected requirements changes
- [ ] Can optimize performance effectively
- [ ] Can write clean, maintainable code under pressure

**Target**: Master frontend component patterns and problem-solving for confident live coding interviews