# Signal to Demux Rebranding Plan (Focused Approach)

This document outlines a targeted plan for renaming "Signal" to "Demux" **only in user-exposed elements** while maintaining internal code structures for compatibility with the upstream project.

## Current State

The project currently contains numerous references to "Signal," but not all need to be renamed. We will focus on:

- User-facing UI elements
- Documentation for end users
- Public API endpoints and their documentation
- Branding elements and identifiers

## Target State

Create a distinct Demux identity in all user-exposed elements while preserving internal code structures to maintain compatibility with the upstream Signal project.

## Key Considerations

### Fork Compatibility

- Preserving internal package names and class structures simplifies merging future updates from upstream Signal
- Maintaining internal naming reduces the risk of introducing bugs
- Focused renaming reduces the scope of changes, making the project more maintainable

### What to Rename

**User-Exposed Elements (RENAME)**:
- Application name and branding
- User interface text and messages
- Public documentation
- Public API endpoints and their documentation
- Error messages visible to users
- Configuration elements visible in user settings
- Public website and distribution materials

**Internal Elements (PRESERVE)**:
- Package names (e.g., `org.whispersystems.textsecuregcm`)
- Internal class names (e.g., `SignalServiceClient`)
- Variable names and method names
- Database schema names (if not directly exposed)
- Internal comments and documentation
- Test cases and internal fixtures

## Implementation Plan

### 1. Inventory of User-Exposed Elements

- Identify all user-visible text in the application
- Catalog all public API endpoints and documentation
- List all branding elements (logos, app names, etc.)
- Identify configuration settings visible to users

### 2. Testing Infrastructure Preparation

- Set up testing focused on user-facing functionality
- Create test cases for API compatibility
- Develop UI tests to ensure rebranded elements appear correctly

### 3. User Interface Updates

- Update all user-facing text resources
- Replace logos and branding elements
- Update application names in visible locations
- Modify visible configuration settings

### 4. Public API Wrapper Layer

- Rather than renaming internal API endpoints, create a wrapper layer:
  - Create new Demux-branded endpoints that map to existing Signal endpoints
  - Implement proper redirects for API compatibility
  - Update API documentation to reflect Demux branding

### 5. Documentation Updates

- Update all end-user documentation
- Revise developer guides for public APIs
- Update website and distribution materials
- Create clear documentation about the fork relationship

### 6. Configuration File Updates

- Update configuration keys that appear in user settings
- Modify visible environment variable names
- Keep internal configuration structures

## Technical Implementation Details

### API Wrapper Approach

Instead of renaming internal code, create API wrappers:

```java
// Existing internal endpoint remains unchanged
@Path("/v1/signal/messages")
public class SignalMessageController {
    // Original implementation
}

// New wrapper endpoint with Demux branding
@Path("/v1/demux/messages")
public class DemuxMessageController {
    @Inject
    private SignalMessageController signalController;
    
    // Delegate to original implementation
    @GET
    public Response getMessages() {
        return signalController.getMessages();
    }
}
```

### User Interface Text Updates

Replace text resources while preserving internal references:

```xml
<!-- Before -->
<string name="app_name">Signal</string>
<string name="notification_message">New Signal message</string>

<!-- After -->
<string name="app_name">Demux</string>
<string name="notification_message">New Demux message</string>
```

## High-Risk Areas

### API Compatibility

- Ensure both Signal and Demux API endpoints work correctly
- Document API evolution strategy for developers
- Test thoroughly with real clients

### User Experience Consistency

- Ensure all visible references are consistently updated
- Avoid mixed branding that could confuse users
- Test with users to identify any missed elements

## Testing Plan

### User Interface Testing

- Visual inspection of all screens
- Automated UI tests for brand consistency
- User acceptance testing

### API Testing

- Test both original Signal endpoints and new Demux endpoints
- Verify that clients can seamlessly use either endpoint
- Performance testing to ensure wrapper approach doesn't impact speed

## Roll-out Strategy

### Phased Approach

1. **Branding Updates**: Replace logos, app names, and obvious visual elements
2. **Documentation**: Update all user-facing documentation
3. **API Extensions**: Add Demux-branded API endpoints alongside Signal endpoints
4. **User Communication**: Clearly communicate changes to users

## Documentation and Communication

### User Communication

- Explain the relationship to Signal in user documentation
- Create clear migration guides for users coming from Signal
- Be transparent about which components are rebranded and which remain the same

## Post-Implementation Verification

- User experience reviews to ensure consistent branding
- API tests to verify all endpoints function correctly
- Verify documentation completely reflects Demux branding

## Maintenance Strategy

- Document how to handle upstream Signal changes
- Create process for merging updates while maintaining Demux branding
- Establish regular review to ensure branding remains consistent as features evolve

## Conclusion

This focused rebranding plan emphasizes changing user-visible elements to establish the Demux identity while preserving internal code structures for compatibility with upstream Signal. This approach balances creating a distinct product identity with the practical benefits of maintaining fork compatibility.

**Benefits of this approach**:
- Reduced development effort compared to full renaming
- Lower risk of introducing bugs
- Easier to incorporate future updates from Signal
- Clear Demux identity for users
- Simplified maintenance 