# Mobile Architecture

## Profile

### What It Is
Mobile Architecture encompasses the patterns and decisions for structuring mobile applications — including platform choice (native, cross-platform, hybrid), app architecture (MVVM, MVI, Clean Architecture), state management, navigation, offline-first design, and platform-specific considerations.

### Origin and Evolution
- Native development: iOS (Objective-C/Swift), Android (Java/Kotlin)
- Hybrid: Cordova/PhoneGap (web in native shell)
- Cross-platform: React Native (2015), Flutter (2018), Kotlin Multiplatform
- Architecture evolution: MVC → MVP → MVVM → MVI → Clean Architecture
- Current: Jetpack Compose (Android), SwiftUI (iOS), declarative UI paradigm

### Key Authors and References
- **Google** — Android Architecture Components, Jetpack Compose
- **Apple** — SwiftUI, Combine framework
- **Facebook/Meta** — React Native
- **Robert C. Martin** — Clean Architecture applied to mobile

### Key Concepts at a Glance
| Approach | Pros | Cons |
|----------|------|------|
| Native | Best performance, full platform access | 2 codebases, 2 teams |
| React Native | JavaScript/TS, one codebase, hot reload | Bridge overhead, native module complexity |
| Flutter | Fast UI, one codebase, custom rendering | Large app size, Dart ecosystem smaller |
| KMP | Shared business logic (Kotlin), native UI | Kotlin expertise required, newer ecosystem |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Offline-first design** — Mobile networks are unreliable. Design for offline operation with sync when connected.
2. **Separation of UI from business logic** — UI frameworks change (XML→Compose, UIKit→SwiftUI). Business logic should survive platform evolution.
3. **Reactive state management** — UI reacts to state changes. Unidirectional data flow prevents state inconsistency.
4. **Platform conventions matter** — Users expect platform-native behavior (back button, gestures, navigation patterns). Don't fight the platform.
5. **Battery and data are constrained** — Optimize for network calls, background processing, and memory usage. Mobile resources are limited.

### When to Use vs. When to Avoid

**Native**: Performance-critical apps, deep platform integration, AR/VR, small team focused on one platform
**Cross-platform**: Shared business logic, limited team resources, content-heavy apps, internal tools
**PWA**: Content-focused, no app store distribution needed, limited budget

---

## Frameworks and Methodologies

### 1. Clean Architecture for Mobile — step-by-step
1. **Domain layer**: entities, use cases, repository interfaces (no framework dependencies)
2. **Data layer**: repository implementations, API clients, local database (Room, Core Data)
3. **Presentation layer**: ViewModels, UI state, platform-specific UI (Compose, SwiftUI)
4. Dependencies point inward: UI → ViewModel → Use Case → Repository
5. Dependency injection: Hilt (Android), Swinject (iOS)
6. Testing: unit test domain/data layers, UI test with fake data

### 2. Offline-First Design — step-by-step
1. Local database is the source of truth (SQLite, Room, Core Data)
2. UI reads from local database only
3. Sync service writes remote changes to local database
4. User actions are queued locally, synced when online
5. Handle conflicts (last-write-wins or user-resolved)
6. Indicate sync status in UI (synced, pending, conflict)

---

## How to Apply

### Decision Checklist
- [ ] Is business logic separated from UI framework?
- [ ] Is offline capability designed in (not bolted on)?
- [ ] Is state management unidirectional and predictable?
- [ ] Are navigation patterns following platform conventions?
- [ ] Is network usage optimized (batching, caching, compression)?

### Implementation Patterns

**MVVM with Unidirectional Flow (Kotlin/Compose):**
```kotlin
data class OrderListState(
    val orders: List<Order> = emptyList(),
    val isLoading: Boolean = false,
    val error: String? = null,
)

class OrderListViewModel(
    private val getOrders: GetOrdersUseCase,
) : ViewModel() {
    private val _state = MutableStateFlow(OrderListState())
    val state = _state.asStateFlow()

    fun loadOrders() {
        viewModelScope.launch {
            _state.update { it.copy(isLoading = true) }
            getOrders()
                .onSuccess { orders -> _state.update { it.copy(orders = orders, isLoading = false) } }
                .onFailure { e -> _state.update { it.copy(error = e.message, isLoading = false) } }
        }
    }
}

@Composable
fun OrderListScreen(viewModel: OrderListViewModel) {
    val state by viewModel.state.collectAsState()
    when {
        state.isLoading -> LoadingIndicator()
        state.error != null -> ErrorMessage(state.error!!)
        else -> OrderList(state.orders)
    }
}
```

### Common Mistakes
1. **Business logic in UI** — ViewModel/Controller contains API calls, validation, transformation
2. **No offline support** — App is useless without network; users expect basic offline functionality
3. **Ignoring platform conventions** — Custom navigation that breaks Android back button or iOS swipe
4. **Memory leaks** — Not managing lifecycle, holding references to destroyed activities/views
5. **One codebase assumption** — Choosing cross-platform without evaluating if native is better for the use case

### Integration with Other Topics
- **Clean Architecture** — Applied to mobile with platform-specific outer layers
- **Design Systems** — Mobile design system for consistent UI components
- **API Design** — Mobile-optimized APIs (BFF, GraphQL for flexible queries)
- **Offline/Sync** — Local-first data architecture
- **Testing** — Unit tests for logic, UI tests for screens, E2E for flows
- **TypeScript** — React Native development with TypeScript

---

## Resources

### Essential Reading
- Android Developer documentation (developer.android.com)
- Apple Developer documentation (developer.apple.com)
- *Clean Architecture for Android* — various community resources

### Online Resources
- Android Architecture Guide (developer.android.com/topic/architecture)
- Swift UI tutorials (developer.apple.com/tutorials/swiftui)
- React Native documentation (reactnative.dev)

### Practice Exercises
1. Structure a mobile app with Clean Architecture layers
2. Implement offline-first data with local database and sync
3. Build a screen with MVVM and unidirectional state flow
4. Compare the same feature in native vs. cross-platform
5. Implement deep linking and navigation following platform conventions
