# Updated Agent Configuration - Subagent Delegation

## What Changed

The uncommitted review system has been updated to use a **delegation pattern** where the main agent automatically invokes specialized subagents for deep security and performance analysis.

## Agent Hierarchy

```
@opencode-agent-uncommitted-review (Main Agent)
    ├── @opencode-agent-security-review (Subagent)
    └── @opencode-agent-performance-review (Subagent)
```

## Key Changes

### 1. Main Agent (`opencode-agent-uncommitted-review.md`)

**Added Delegation Logic**:
- Now instructs LLM when to delegate to @security-review and @performance-review
- Includes clear criteria for invoking each subagent
- Integrates subagent results into overall assessment

**New Sections**:
- **When to Delegate to Subagents**: Detailed decision tree
- **Updated Review Process**: Step 2 now includes delegation
- **Enhanced Overall Assessment**: Shows which subagents were invoked
- **Modified "What to Check"**: Explicit delegation markers

**Behavior**:
- Automatically invokes @security-review for authentication, user input, database queries, crypto operations
- Automatically invokes @performance-review for loops, heavy computations, database ops, file I/O
- Skips delegation for minimal, cosmetic, or configuration changes
- Remains responsible for final "Ready to commit" decision

### 2. Security Subagent (`opencode-agent-security-review.md`)

**Configuration Changes**:
- Added `hidden: true` to frontmatter
- Won't appear in user autocomplete menu
- Can still be invoked by main agent via Task tool

**Purpose**:
- Deep security vulnerability analysis
- Focused on injection attacks, auth flaws, data exposure
- Returns structured security assessment with CVSS scores

### 3. Performance Subagent (`opencode-agent-performance-review.md`)

**Configuration Changes**:
- Added `hidden: true` to frontmatter
- Won't appear in user autocomplete menu
- Can still be invoked by main agent via Task tool

**Purpose**:
- Performance bottleneck analysis
- Algorithm complexity review
- Caching, database, and I/O optimization recommendations

## How It Works

### User Experience

User simply runs:
```
@uncommitted-review
```

### Internal Flow

1. **Main agent starts**
2. **Checks git diff** to understand changes
3. **Evaluates delegation criteria**:
   - Does this involve security-sensitive operations? → Invoke @security-review
   - Does this involve performance-sensitive operations? → Invoke @performance-review
4. **Invokes subagents** (if criteria met) in parallel
5. **Analyzes code itself** for general issues
6. **Integrates all findings** into cohesive review
7. **Presents combined assessment** with subagent statuses

### Example Output

```markdown
## Summary
Modified 3 files in authentication module. Changes add JWT validation and session management.

## Critical Issues
[Issues from main agent's review]

## High Priority Issues
[Issues from @performance-review integrated here]

## Medium Priority Issues
[Issues from all sources]

## Low Priority Issues
[Code style from main agent]

## Overall Assessment
- Quality Score: 7/10
- Security Score: 8/10 (from @security-review)
- Performance Score: 6/10 (from @performance-review)
- Ready to commit: No
- Required actions: Fix session token leak and optimize database query

---
**Note**: This review was enhanced by specialized subagents:
- @security-review: invoked
- @performance-review: invoked
```

## Delegation Criteria

### When to Invoke @security-review

Main agent will invoke when it detects:
- User input handling or validation
- Authentication/authorization code
- Database queries with user data
- File operations with user paths
- Cryptographic operations
- Secret/credential management
- API endpoints or network services
- Any code processing untrusted input

### When to Invoke @performance-review

Main agent will invoke when it detects:
- Nested loops or iterations
- Database operations
- Large data processing
- Frequent API calls
- Memory-intensive operations
- Concurrent/parallel code
- File system operations

### When to Skip Delegation

Main agent will skip subagents when:
- Changes are minimal (< 50 lines)
- Code is configuration or docs only
- Changes are cosmetic (whitespace, formatting)
- User explicitly requests quick review

## Benefits

### 1. Comprehensive Reviews
- Security expert analyzes vulnerabilities deeply
- Performance expert finds optimization opportunities
- Main agent provides holistic view

### 2. Intelligent Automation
- Automatic delegation based on code analysis
- No manual @mentioning needed
- Users get comprehensive reviews without effort

### 3. Expert-Level Analysis
- Each agent is highly specialized
- Prompts tuned for specific focus areas
- Better than generalist approach

### 4. Scalability
- Subagents can be invoked independently if needed
- Easy to add more specialized agents (testing, accessibility, etc.)
- Main agent orchestrates seamlessly

### 5. User-Friendly
- Single command invocation
- Clear indication of which analyses were performed
- Integrated results, not fragmented outputs

## Manual Override

Users can still invoke subagents directly if needed:

```
@security-review Focus on this specific function
@performance-review Check database queries only
```

However, the typical workflow is just:
```
@uncommitted-review
```

## Technical Details

### Frontmatter Configuration

**Main Agent**:
```yaml
mode: subagent
temperature: 0.1
# No 'hidden' - visible to users
```

**Subagents**:
```yaml
mode: subagent
hidden: true  # Not in user autocomplete
temperature: 0.1
```

### Permissions

All agents have identical permissions:
- Read-only file access (write/edit denied)
- Git commands allowed (git diff*, git log*, etc.)
- Webfetch denied
- Task permission allows subagent invocation

### Model Usage

All agents use OpenCode's default model (no model specified).

## Testing the Configuration

### 1. Restart OpenCode
```bash
# Exit and restart to load changes
opencode
```

### 2. Test with Simple Change
```bash
# Make small change
echo "// test" >> app.js

# Run review
@uncommitted-review
```

**Expected**: Should skip subagent delegation for small change

### 3. Test with Security-Sensitive Code
```bash
# Add user input handling
echo "const userInput = req.body;" >> app.js

# Run review
@uncommitted-review
```

**Expected**: Should invoke @security-review automatically

### 4. Test with Performance-Sensitive Code
```bash
# Add nested loops
cat > app.js << 'EOF'
for (i = 0; i < n; i++) {
  for (j = 0; j < m; j++) {
    process(i, j);
  }
}
EOF

# Run review
@uncommitted-review
```

**Expected**: Should invoke @performance-review automatically

## Troubleshooting

### Subagents Not Invoking

**Symptoms**: Main agent runs but no subagent delegation

**Possible Causes**:
1. Agent files not loaded (restart OpenCode)
2. Code changes don't meet delegation criteria
3. LLM decides not to delegate (expected behavior for some changes)

**Solutions**:
1. Verify agents are in correct directory: `~/.config/opencode/agent/`
2. Check file names match exactly
3. Restart OpenCode completely
4. Try with more complex security/performance code

### Missing Security/Performance Scores

**Symptoms**: Output doesn't show security/performance scores

**Expected Behavior**:
- Scores appear only when subagents are invoked
- If changes don't trigger delegation, scores won't appear

### Subagents Visible in Autocomplete

**Symptoms**: @security-review and @performance-review appear in menu

**Cause**: `hidden: true` not set or not loaded

**Solution**: Verify frontmatter has `hidden: true` and restart OpenCode

## Extending the System

### Adding More Specialized Agents

To add more subagents (e.g., @accessibility-review, @test-coverage-review):

1. Create new agent file:
   ```yaml
   ---
   description: Focus on [domain]
   mode: subagent
   hidden: true
   temperature: 0.1
   permission: [same as others]
   ---
   ```

2. Update main agent to delegate:
   ```markdown
   ### Invoke @new-agent when:
   - [criteria for invocation]
   ```

3. Update output format:
   ```markdown
   - [Domain] Score: X/10 (from @new-agent if invoked)
   ```

### Modifying Delegation Logic

To make delegation more aggressive or conservative:

Edit main agent's "When to Delegate to Subagents" section:
- Lower thresholds for more delegation
- Raise thresholds for less delegation
- Add or remove specific criteria

## Comparison: Before vs After

### Before
- User invokes main agent only
- General code quality review
- Deep security/performance requires manual invocation
- Results fragmented across multiple agents

### After
- User invokes main agent only
- Automatic intelligent delegation
- Integrated comprehensive review
- Single cohesive output
- Clear indication of what was analyzed

## Performance Considerations

- Subagent invocations use additional tokens
- For trivial changes, delegation is skipped
- Parallel invocation when multiple subagents needed
- Overall review time increases with subagent delegation
- Trade-off: Better analysis vs faster reviews

## Future Enhancements

Potential improvements:
1. **User preferences**: Configurable delegation thresholds
2. **Project-specific rules**: Custom delegation criteria per repo
3. **Caching**: Remember which files need deep analysis
4. **Progressive disclosure**: Show subagent results as they arrive
5. **Interactive**: Ask user if they want deep analysis

## Conclusion

The updated agent system provides:
- ✅ Automatic, intelligent delegation
- ✅ Expert-level specialized analysis
- ✅ Integrated, cohesive output
- ✅ Simple user experience
- ✅ Extensible architecture

Users get comprehensive reviews with a single command, while agents specialize in their domains of expertise.
