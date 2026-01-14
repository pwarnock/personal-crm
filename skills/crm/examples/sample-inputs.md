# Sample Inputs and Expected Processing

## Event Capture Examples

### Simple Meetup
**Input**: "met Brian and Greg for coffee yesterday"
**Extracted**:
- Event name: "coffee"
- Date: yesterday (relative to today)
- Attendees: Brian, Greg

**Action**: Search contacts for "Brian" and "Greg", create event with resolved attendees.

---

### Named Event
**Input**: "jsc-4875 breakfast last Saturday with Greg, Brian, Peter, Renee, Alice, and Cosette"
**Extracted**:
- Event name: "jsc-4875 breakfast"
- Date: last Saturday
- Attendees: Greg, Brian, Peter, Renee, Alice, Cosette

**Action**: Search and resolve all 6 attendees, create event.

---

### With Location
**Input**: "lunch with Alice at Blue Bottle on Tuesday"
**Extracted**:
- Event name: "lunch at Blue Bottle"
- Date: Tuesday (next Tuesday if today is past Tuesday)
- Attendees: Alice

---

### Voice Transcript (Messy)
**Input**: "um so I had breakfast with uh Brian and Greg this morning we talked about the project"
**Extracted**:
- Event name: "breakfast"
- Date: this morning (today)
- Attendees: Brian, Greg
- Notes: "talked about the project"

---

## Contact Creation Examples

### New Contact
**Input**: "met Sarah Chen at the conference, she's a PM at Stripe"
**Extracted**:
- Name: Sarah Chen
- Company: Stripe
- Role: PM
- Context: met at conference

**Action**: Create new contact, optionally create event for "conference meeting".

---

### Multiple New Contacts
**Input**: "networking event yesterday, met John from Google and Maria from Meta"
**Extracted**:
- Event: "networking event", yesterday
- New contacts:
  - John (Google)
  - Maria (Meta)

---

## Ambiguity Resolution Examples

### Multiple Matches
**Input**: "coffee with Brian"
**Search result**: Brian Arizaga, Brian Pollard

**Resolution**: Ask user "Which Brian?" with options:
1. Brian Arizaga
2. Brian Pollard
3. Both
4. Other (type name)

---

### No Match
**Input**: "dinner with Alex"
**Search result**: No "Alex" found

**Resolution**: Ask user:
1. Create new contact "Alex"
2. Search by different name
3. Skip this attendee

---

## Date Parsing Examples

| Input | Parsed Date |
|-------|-------------|
| "yesterday" | (today - 1 day) |
| "last Saturday" | Most recent past Saturday |
| "this morning" | Today |
| "Tuesday" | Next Tuesday (or this Tuesday if not yet passed) |
| "Jan 10" | January 10, current year |
| "1/10" | January 10, current year |
