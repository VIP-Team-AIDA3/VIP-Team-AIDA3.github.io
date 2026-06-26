# Mini-Tutorial 3A - Air Traffic Control Basics

*What is air traffic control, why does it exist, and what do we need it for?*

Air traffic control is the service that keeps aircraft apart and keeps them moving. It is the reason that dozens of aircraft can share the same sky and the same runways without colliding, even though each pilot can see only a small part of what is happening around them.

In this tutorial we will build the idea up from the ground. We will start with the problem that makes the service necessary, state what air traffic control actually is, separate it from the broader idea of air traffic management, and look at the services and the people who provide them. We will then spend most of our time on the part that matters most for anyone building language systems for aviation: the controlled language that controllers and pilots use, where it came from, and why it still fails. Two runway collisions, almost fifty years apart, run through that final part.

## Why We Need Air Traffic Control

Flight is hard to coordinate for three reasons that all act at once.

- **Aircraft are fast.** Two aircraft can close on each other at several hundred knots. There is very little time to notice a conflict and react to it.

- **Pilots cannot see most other traffic.** The view from a cockpit is narrow, and cloud or darkness can remove it entirely. Pilots cannot keep themselves apart by eyesight alone, especially when traffic is heavy.

- **Airspace and runways are shared and finite.** Many aircraft want the same routes, altitudes, and runways at the same moment. Something has to decide the order.

A ground-based service solves this by holding one shared picture of all the traffic and issuing authority for who goes where and when. This is just as true for uncrewed aircraft. For example, a Windracers UAV operating in controlled airspace receives clearances and instructions from a controller exactly as a crewed aircraft does, and the operator must answer in the same controlled language.

### Example: Two Aircraft, One Runway

Suppose two aircraft are near the same airport. One is on final approach to land on runway 22 right. The other is holding short of the same runway, waiting to depart. Only one aircraft can use the runway at a time, and a mistake here is a runway incursion, one of the most serious events in aviation.

The controller resolves this with two separate authorizations, issued in an order that keeps the runway clear:

| Speaker | Transmission |
| --- | --- |
| **Tower** | Purdue 492, hold short of runway 22 right, landing traffic. |
| **Purdue 492** | Holding short of runway 22 right, Purdue 492. |
| *(the arriving aircraft lands and clears the runway)* | |
| **Tower** | Purdue 492, runway 22 right, cleared for takeoff. |
| **Purdue 492** | Cleared for takeoff runway 22 right, Purdue 492. |

Nothing was left to the pilots to negotiate between themselves. The controller held the full picture, decided the order, and issued each instruction one at a time. The pilot repeated each one back so the controller could confirm it was heard correctly. This small exchange is air traffic control in miniature.

## What Air Traffic Control Is

The International Civil Aviation Organization (ICAO) defines an air traffic control service as a service provided for two purposes: preventing collisions, and expediting and maintaining an orderly flow of air traffic [1], [2]. The collision-prevention purpose itself has two parts: keeping aircraft apart from one another in the air, and keeping aircraft apart from other aircraft and from obstructions on the movement area of an airport [1].

Two boundaries are commonly misunderstood and worth fixing now.

- The stated objectives do not include preventing collisions with the ground or with high obstacles. That responsibility stays with the pilot [1].

- A clearance authorizes an aircraft to proceed only with respect to known traffic. It never overrides a safety rule, and it never transfers the pilot's responsibility to the controller [1].

So air traffic control does not fly the aircraft. It issues **clearances** and **instructions** to a pilot, who stays in command.

**Key terms**

- **Clearance:** authorization for an aircraft to proceed under stated conditions [1].
- **Instruction:** a directive requiring a pilot to take a specific action [1].
- **Controlled flight:** any flight that is subject to an air traffic control clearance [1].
- **Separation:** the minimum spacing kept between aircraft.

## Air Traffic Control and Air Traffic Management

It is easy to treat "air traffic control" and "air traffic management" as the same thing. They are not. Air traffic management (ATM) is the larger idea, and air traffic control (ATC) is one component inside it.

ICAO defines air traffic management as the dynamic, integrated management of air traffic and airspace, carried out safely, economically, and efficiently [1]. ATM has three parts:

- **Air traffic services (ATS):** the services provided directly to flights, which include air traffic control along with flight information, alerting, and advisory services.
- **Airspace management (ASM):** how airspace is structured and allocated among different users.
- **Air traffic flow management (ATFM):** regulating the volume of traffic so that demand does not exceed the capacity a control unit can safely handle [1].

The relationship nests. Air traffic control sits inside air traffic services, and air traffic services sit inside air traffic management, next to airspace management and flow management.

| Term | Scope | Question it answers |
| --- | --- | --- |
| ATM | The whole system | How are air traffic and airspace managed overall? |
| ATS | Services to flights | What services does a given flight receive? |
| ATC | One of those services | Who is cleared to go where, right now? |
| ATFM | Flow regulation | Is demand within the available capacity? |
| ASM | Airspace allocation | How is the airspace divided and shared? |

A short way to hold the distinction: air traffic control is **tactical** and works in seconds, keeping these two aircraft apart right now. Air traffic management is **strategic** and works over minutes to months, making sure the system as a whole has the structure and capacity for that control to be possible. The rulebook that governs procedures, ICAO Doc 4444, is itself titled *Procedures for Air Navigation Services - Air Traffic Management*, because controlling individual aircraft is one activity within managing the larger system [1].

## The Services and the Units

Air traffic control is one member of the wider set of air traffic services. Under ICAO, "air traffic services" is a general term that covers several distinct services [1], [3]:

- **Air traffic control service:** prevents collisions and keeps traffic orderly. This is the only service that issues clearances.
- **Flight information service:** provides information useful for safe and efficient flight, such as weather and traffic information, without issuing clearances.
- **Alerting service:** notifies search and rescue organizations when an aircraft is in difficulty, and assists them.
- **Air traffic advisory service:** offers advice on collision hazards in certain airspace, but only suggests rather than clears [1].

The distinction between a clearance and advice is the important one. Advisory and information services use words like "advise" or "suggest." Only the control service authorizes action with a clearance [1].

Air traffic control service is delivered by three kinds of unit, each matched to a phase of flight [1], [3]:

| Unit | Service it provides | Phase of flight |
| --- | --- | --- |
| Aerodrome control tower | Aerodrome control service | On and around the airport: taxi, takeoff, landing |
| Approach control | Approach control service | Climbing out after departure, descending in to arrive |
| Area control centre (ACC) | Area control service | En route, cruising between airports |

A single flight is handed from one unit to the next as it progresses: tower to approach to area control on the way up, and the reverse on the way down. Each handoff is itself a short radio exchange, which we will see below.

Air traffic control is not provided the same way everywhere. Airspace is divided into classes labeled A through G, and the class determines what service an aircraft receives and what is required to enter it [2], [3]. The classification is an international scheme set out in ICAO Annex 11, and each country implements it with its own dimensions and procedures. Controlled airspace, where an air traffic control service is provided according to the class, covers Classes A through E [3]. The remaining airspace carries lighter service, such as flight information rather than full control. In short, the class of airspace tells a pilot, or a UAV operator, what to expect from the ground and what to do before entering.

### The Classes of Airspace

The classes can be read as a scale, from the most tightly controlled to the least. The figure below shows the United States implementation. It is useful here because, alongside the altitude limits, it marks what a small uncrewed aircraft must do in each class.

<div align="center"><img width="624" height="252" alt="AirspaceClass" src="https://github.com/user-attachments/assets/47e60286-86a3-4323-b262-ea36e3257ca2" /></div>

*Fig. 1 Airspace classes and their altitude limits, annotated for small UAS operators. The controlled classes (B, C, D, and the surface area of E) require air traffic authorization before flight, while Class G does not. Source: Federal Aviation Administration [11].*

| Class | Where it is, and typical limits | What it takes to enter |
| --- | --- | --- |
| **A** | 18,000 ft MSL up to flight level 600, across the whole country | Instrument flight only, under continuous ATC control. No visual-flight traffic. |
| **B** | Around the busiest airports, surface up to roughly 10,000 ft MSL, shaped like an upside-down wedding cake | An explicit ATC clearance to enter. |
| **C** | Around moderately busy airports with a control tower and radar, surface to about 4,000 ft AGL | Two-way radio contact established with ATC before entering. |
| **D** | Around airports with an operational control tower, surface to about 2,500 ft AGL | Two-way radio contact established with ATC before entering. |
| **E** | Controlled airspace that is not A, B, C, or D. Floors vary: commonly 700 or 1,200 ft AGL, 14,500 ft MSL in places, or the surface near certain airports | Visual flights need no clearance in most of it; instrument flights are controlled throughout. |
| **G** | Uncontrolled airspace, from the surface up to the base of the overlying Class E | Nothing from ATC. Pilots keep themselves apart by see-and-avoid. |

Classes A through E are controlled, which means an air traffic control service is provided there. Class G is uncontrolled: ATC neither clears nor separates traffic in it, although a flight information service may still be available [3].

For a small uncrewed aircraft the picture is simpler but stricter. In controlled airspace (Classes B, C, D, and the surface area of Class E) a UAV operator must obtain air traffic authorization before flying, which in the United States is handled through automated systems rather than a live radio call. In uncontrolled Class G no authorization is required. In all classes, small UAS operations are generally held at or below 400 ft above ground level unless a waiver is granted [11]. This low layer is where most UAV operations sit, and it is one reason the communication and authorization handling described in this tutorial carries over directly to uncrewed flight.

## How Controllers and Pilots Talk

Air traffic control runs mostly over voice radio on shared frequencies, so the language itself is part of the safety design. ICAO defines a closed set of standard words and phrases, each with a single fixed meaning, so that a word means the same thing to every controller and pilot in the world [3], [4]. This controlled language is, in effect, a constrained version of English built for one purpose, and it is the common working language of international aviation [10]. A few of the most important words:

| Word | Meaning |
| --- | --- |
| CLEARED | Authorized to proceed under the conditions specified |
| AFFIRM | Yes |
| NEGATIVE | No, or permission not granted, or not correct |
| ROGER | I have received all of your last transmission |
| WILCO | I understand and will comply |
| STANDBY | Wait and I will call you |
| SAY AGAIN | Repeat your last transmission |
| READ BACK | Repeat this message back to me exactly as received |

Plain conversational words such as "yes," "okay," and "go ahead" are avoided or banned outright, because they are ambiguous over a noisy radio [3], [4].

### Anatomy of a Transmission

A transmission follows a fixed order so that the listener knows immediately who is talking to whom. A controller begins with the aircraft callsign, then the instruction. A pilot begins with the receiving station, then their own callsign, then the message.

| Speaker | Transmission | Structure |
| --- | --- | --- |
| **Pilot** | Kennington Tower, Purdue 492, request taxi. | receiving station, own callsign, message |
| **Tower** | Purdue 492, taxi to holding point runway 22 right via taxiway Bravo. | callsign, instruction |
| **Purdue 492** | Taxi to holding point runway 22 right via Bravo, Purdue 492. | readback, own callsign last |

The pilot ends the readback with the callsign. This tells the controller, and every other aircraft listening on the frequency, exactly which aircraft accepted the instruction.

## Types of Transmission

Almost every transmission on the frequency is doing one of a few jobs. Recognizing the type tells you who is expected to act and what the reply should be.

| Type | Who sends it | What it conveys | Example | Expected response |
| --- | --- | --- | --- | --- |
| **Instruction** | Controller to pilot | A required action. It is a directive, phrased as a command, that the pilot must carry out or decline with "unable" [1]. | "Purdue 492, turn right heading 250." | Read back the action, callsign last. |
| **Clearance** (an authorization) | Controller to pilot | Permission to proceed under stated conditions. It grants rather than directs, but within its terms it must be obeyed [1]. | "Purdue 492, cleared for takeoff runway 22 right." | Read back in full, callsign last. |
| **Request** | Usually pilot to controller | A wish to obtain something: a clearance, an action, or information [3], [4]. | "Purdue 492, request descent." | The controller approves, issues an instruction, or replies "standby" or "unable." |
| **Information** | Either direction | Facts that do not by themselves require an action: traffic, weather, runway condition, a frequency, ATIS. | "Purdue 492, traffic is a Cessna, 2 o'clock, 3 miles, opposite direction." | Acknowledge, for example "Windracers 492, looking" or "traffic in sight." |
| **Report** | Pilot to controller, often when asked | Specified information about the aircraft's position or status [3], [4]. | "Purdue 492, passing flight level 120." | The controller acknowledges, or acts on it. |

A quick test for the type: an instruction or clearance changes what the aircraft does, a request asks for one, information describes the world, and a report describes the aircraft.

### What Every Transmission Must Contain

Two elements are never optional.

- **Callsign.** Every transmission identifies the station it is for and the station it is from. On first contact, the pilot names the station being called, then gives the aircraft's own callsign: "Kennington Tower, Purdue 492." Once two-way contact is established, the called station's name can be dropped, but the aircraft callsign stays on every transmission, so it is always clear who is speaking and who is being addressed [3], [4].

- **Content matched to the type.** The body carries the elements that type needs, in standard phraseology. An instruction states the action and its value, with numbers spoken to the convention, for example a heading as three digits. A request uses the word "request" and names what is wanted. A report passes the item asked for, often prompted by the controller's word "report."

For an instruction or a clearance, a third element is added on the receiving side: a **readback** of the safety-critical parts, ending with the callsign [1], [3]. The readback is what closes the loop and turns a one-way message into a confirmed, two-way agreement.

A worked contrast:

| | Missing a required element | Complete |
| --- | --- | --- |
| Instruction readback | "Right heading 250." | "Right heading 250, Purdue 492." |
| Request | "Request lower." | "Purdue 492, request descent flight level 80." |

In each case the second version can be acted on with confidence, because the controller knows exactly which aircraft is speaking and exactly what is meant.

### Example: A Departure Clearance

A route clearance can be long. It must still be read back in full, because every element matters.

| Speaker | Transmission |
| --- | --- |
| **Tower** | Purdue 123, cleared to Kennington via Alpha 1, flight level 280, Wicken 3 Delta departure, squawk 5501. |
| **Purdue 123** | Cleared to Kennington via Alpha 1, flight level 280, Wicken 3 Delta departure, squawk 5501, Purdue 123. |

The **squawk** is the four-digit code the pilot sets on the aircraft transponder so the controller's radar display can identify and track the aircraft. A route clearance like this is not permission to take off or to enter the runway. The words "take off" are reserved for the actual takeoff clearance, to remove any chance of confusion [3].

### Example: A Frequency Handoff

When a flight moves from one unit's area to the next, the first unit hands it over.

| Speaker | Transmission |
| --- | --- |
| **Approach** | Purdue 492, contact Kennington Tower 118.9. |
| **Purdue 492** | 118.9, Purdue 492. |

The pilot reads back the new frequency so a mishearing does not leave the aircraft talking to no one.

### Readback: the Safety Loop

Readback is the single most important safety feature in voice communication. Certain safety-critical items must always be repeated back by the pilot exactly as received, so the controller can confirm two things: that the message was heard correctly, and that the right aircraft is acting on it [1], [3]. Items that must always be read back include route clearances, any instruction to enter, cross, or take off from a runway, the runway in use, altimeter settings, transponder codes, and level, heading, and speed instructions [1], [3].

The loop earns its value when it catches a mistake:

| Speaker | Transmission |
| --- | --- |
| **Control** | Purdue 321, descend flight level 80. |
| **Purdue 321** | Descend flight level 180, Purdue 321. |
| **Control** | Purdue 321, negative. Descend flight level 80, eight zero. |
| **Purdue 321** | Descend flight level 80, Purdue 321. |

The pilot misheard "80" as "180." Without readback, the aircraft would have stopped its descent 10,000 feet too high, possibly into other traffic. The readback exposed the error in seconds and the controller corrected it. A one-way instruction became a verified, two-way agreement.

## Why the Language Is Standardized: Two Collisions

The strict language in the previous section can feel excessive until you see what its absence costs. Two runway collisions, almost fifty years apart, show both why the standard was built and why it still has to be enforced.

### Tenerife, 1977: The Accident That Built the Rules

<div align="center"><img width="768" height="280" alt="Tenerife-airport-disaster-crashsite-overview-german" src="https://github.com/user-attachments/assets/67dc9072-e733-46c1-86c8-dae9973cc2dc" /></div>

On 27 March 1977, two Boeing 747s collided on the runway at Los Rodeos Airport in Tenerife. 583 people died, and it remains the deadliest accident in aviation history [8].

Fog had reduced visibility, and both aircraft were taxiing on the single runway. The KLM aircraft reached the takeoff end, turned into position, and its crew believed it had permission to depart. The first officer transmitted a non-standard phrase, reporting that they were "now at takeoff," wording that existed in neither the controller nor the pilot vocabulary [9]. The tower had not issued a takeoff clearance. At nearly the same moment, the Pan Am crew transmitted that they were still on the runway. The two transmissions reached the tower at once and interfered, producing a squeal that blocked the warning [9]. The KLM aircraft was already accelerating. It struck the Pan Am aircraft, which was still on the runway.

The accident was not caused by weather or by a failed engine. It was caused by language, and by a shared understanding that turned out to be false. The response reshaped how aviation talks:

- The word "takeoff" was restricted to the actual takeoff clearance. At every other stage, controllers and pilots say "departure" instead, so the word that authorizes a roll cannot appear by accident in ordinary conversation [3], [6].
- Readback of key items became the expected practice. An instruction is not acknowledged with "OK," or even with "Roger," which only means a transmission was received. The pilot repeats the safety-critical parts back, so a false understanding is exposed before it is acted on [3], [6].
- Standard ICAO phraseology and the use of English as a common working language were reinforced worldwide [6], [10].
- Crew resource management (CRM) was developed so that junior crew members are expected to speak up across the authority gradient. United Airlines launched the first formal program in 1981, and the practice later became mandatory for airline crews [6].

The non-standard phrasing at the center of the accident is worth contrasting directly with the standard phrasing that replaced it:

| Situation | Non-standard (unsafe) | Standard (correct) |
| --- | --- | --- |
| Reporting readiness | "We are now at takeoff." | "Ready for departure." |
| Authorizing departure | "OK, you can go." | "Cleared for takeoff." |

These phrases are not interchangeable. "Ready for departure" is a status report a pilot makes. "Cleared for takeoff" is an authorization that only a controller can give. Keeping the two apart is exactly what stops one aircraft from rolling while another is on the runway.

### LaGuardia, 2026: The Problem Has Not Gone Away

<div align="center"><img width="468" height="584" alt="LaGuardia" src="https://github.com/user-attachments/assets/b9f8bd0b-f18a-4984-b063-ac53bceace5b" /></div>


Standard phraseology has prevented countless accidents, but it has not removed the underlying risk, because the system still depends on people hearing and understanding each other over a shared radio. On 22 March 2026, a regional jet operating as Air Canada Express Flight 8646 was landing at New York LaGuardia when it collided with an airport fire and rescue truck that had been cleared to cross the active runway. Both pilots were killed, and passengers and firefighters were injured. The investigation is led by the National Transportation Safety Board and is ongoing, so the account here rests on preliminary findings [7], [8].

What stands out for our purposes is how closely the communication problems echo Tenerife. According to the NTSB's preliminary reporting, the truck was cleared to cross the runway only seconds before the jet touched down. When the controller recognized the conflict and called for the truck to stop, the driver heard the word "stop" but was not immediately certain the call was meant for his vehicle, because the transmission did not at first identify the addressee unambiguously [7]. Separately, an attempt by one of the responding vehicles to reach the tower was lost when a second transmission arrived on the same frequency at the same moment [7]. A ground radar system that should have warned the controllers did not generate an alert, in part because the vehicles were not fitted with transponders [7], [8].

Two of these failures are the same failures that appeared in 1977: an instruction whose intended recipient was not instantly clear, and a critical transmission lost because two parties spoke on one frequency at once. Standard phraseology, callsign discipline, and readback exist precisely to manage these failure modes, and the accident is a reminder that they have to be applied every time, by everyone on the frequency, ground vehicles included, not only aircraft. It is also part of why the work this tutorial supports matters. A language system that can parse a live transmission and flag an ambiguous or blocked instruction adds a layer of checking that does not depend on a tired human catching the error in three seconds.

## Worked Examples

These exchanges put the pieces together. Read each one, then read the note that follows.

### Example 1: A Full Departure Sequence

| Speaker | Transmission |
| --- | --- |
| **Purdue 1425** | Kennington Ground, Purdue 1425, request taxi, IFR to Marlow. |
| **Ground** | Purdue 1425, taxi to holding point runway 22 right via Bravo, hold short of runway 22 right. |
| **Purdue 1425** | Taxi to holding point 22 right via Bravo, holding short of 22 right, Purdue 1425. |
| **Ground** | Purdue 1425, contact Tower 118.9. |
| **Purdue 1425** | 118.9, Purdue 1425. |
| **Purdue 1425** | Kennington Tower, Purdue 1425, holding short runway 22 right, ready for departure. |
| **Tower** | Purdue 1425, runway 22 right, cleared for takeoff. |
| **Purdue 1425** | Cleared for takeoff runway 22 right, California 1425. |

*Note.* The aircraft is handled by two units, Ground then Tower, with a handoff in between. The pilot reports "ready for departure," not "ready for takeoff," and only the Tower's "cleared for takeoff" authorizes the roll. Every runway instruction is read back.

### Example 2: A Conditional Clearance

Sometimes a controller wants an aircraft to act after another aircraft has passed. This is a conditional clearance, and it has a strict order: identification, the condition, the clearance, then a brief restatement of the condition [1].

| Speaker | Transmission |
| --- | --- |
| **Tower** | Purdue 486, behind the landing Cessna, line up runway 22 right behind. |
| **Purdue 486** | Behind the landing Cessna, lining up 22 right behind, Chicago 486. |

*Note.* The pilot must positively identify the aircraft named in the condition, the landing Cessna, before acting. The word "behind" appears at the start and the end so that the condition cannot be lost. This phrasing is only permitted when both the controller and the pilot can see the aircraft causing the condition [1].

### Example 3: Spot the Problem

Read this exchange and decide what is wrong before reading the note.

| Speaker | Transmission |
| --- | --- |
| **Tower** | Purdue 1820, descend flight level 90, traffic is a Cessna 2 o'clock. |
| **Purdue 1820** | Okay, going down. |

*Note.* The readback is unusable. "Okay, going down" does not repeat the assigned level, so the controller cannot confirm the pilot heard "flight level 90" rather than some other level. It also omits the callsign, so no one knows which aircraft answered. A correct readback would be: "Descend flight level 90, Purdue 1820." This is the same class of failure that the rules in the previous section were written to prevent.

## Summary

Air traffic control exists because aircraft are fast, pilots cannot see most other traffic, and airspace is shared and finite. ICAO defines the service by two purposes: preventing collisions and maintaining an orderly flow [1]. It is one of several air traffic services, and only it issues clearances. It is also one piece of the broader picture of air traffic management, which adds airspace management and flow management on top of the services provided to flights [1]. Control is delivered by aerodrome, approach, and area control units, each matched to a phase of flight, within airspace whose class sets the level of service.

Because the service runs over voice radio, it depends on a closed, standardized language and on readback, the two-way loop that confirms a clearance was both transmitted and received as intended. That standard language was built in response to the 1977 Tenerife collision, where non-standard phrasing and a blocked transmission killed 583 people [8], [9]. The 2026 LaGuardia collision shows that the same failure modes, ambiguous addressing and overlapping transmissions, can still defeat the system, which is why the language has to be applied consistently and why automated checking of live communication is worth building [7].

## Practice Questions

### 1. Build a Readback

A controller transmits: "Windracers 492, turn right heading 250, climb flight level 120." Write the pilot's correct readback.

### 2. Find the Error

A pilot transmits: "Roger, cleared for takeoff." What is missing or wrong, and why does it matter on a shared frequency?

### 3. Order a Conditional Clearance

A controller wants Windracers 492 to line up on runway 22 right only after a departing Piper has passed. Put these four parts in the correct order: the clearance, the condition restated, the aircraft identification, the condition.

### 4. Service or Clearance

For each item, decide whether it is a clearance or an information service: (a) "Cleared to land runway 22 right." (b) "Wind 230 degrees, 12 knots." (c) "Cleared for takeoff." (d) "Traffic is a helicopter, 3 miles north."

### 5. Control or Management

Classify each activity as air traffic control (ATC) or air traffic flow management (ATFM): (a) telling one aircraft to turn left to avoid another; (b) delaying departures across a region because an airport is forecast to exceed its capacity; (c) clearing an aircraft to land; (d) rerouting a traffic flow two months ahead of a busy season.

### 6. The Common Thread

Tenerife in 1977 and LaGuardia in 2026 shared two communication failures. Name them, and give the standard safeguard intended to manage each.

## Wrapping Up

Thank you for working through this module. Air traffic control is a large subject, and you have now covered the parts that have to come first before any of the finer detail makes sense.

Here is what you should be able to do:

- Explain why the service exists: aircraft are fast, pilots cannot see most other traffic, and airspace and runways are shared and finite.
- State the two purposes of air traffic control, preventing collisions and maintaining an orderly flow, along with the boundary that the pilot, not the controller, stays in command.
- Place air traffic control inside the larger picture of air traffic management, next to the other services, airspace management, and flow management.
- Identify the control units, tower, approach, and area control, and match each to a phase of flight.
- Read an airspace class and say what service it carries and what entry requires, including the authorization a small UAV needs.
- Follow a real radio exchange: the fixed order of a transmission, the standard phraseology, and the readback loop that confirms a clearance was heard correctly.
- Explain, from Tenerife and LaGuardia, why the standardized language was built and why it still has to be applied every time, by everyone on the frequency.

If one idea is worth carrying out of this module, it is that the language is the safety system. A clearance only works when both sides share the same understanding of it, and almost everything in air traffic control communication exists to protect that shared understanding.

The questions above are there to test these ideas. Work through them first, then check the answer key below.

## Solutions

**1.** "Right heading 250, climb flight level 120, Windracers 492." Both the heading and the level are read back, and the callsign comes last.

**2.** Two problems. "Roger" must never replace a readback, and there is no callsign. The controller cannot tell which aircraft accepted the takeoff clearance, and on a busy frequency another aircraft might act on it. A correct reply is "Cleared for takeoff runway 22 right, Windracers 492."

**3.** Identification, condition, clearance, restated condition: "Windracers 492, behind the departing Piper, line up runway 22 right behind."

**4.** (a) clearance, (b) information, (c) clearance, (d) information. Only (a) and (c) authorize the aircraft to do something. Items (b) and (d) give the pilot information to use in their own decisions.

**5.** (a) ATC, (b) ATFM, (c) ATC, (d) ATFM. Control acts on individual aircraft in the moment. Flow management acts on volumes of traffic ahead of time so that demand stays within capacity.

**6.** First, an instruction whose intended recipient was not immediately clear; the safeguard is callsign discipline, addressing every transmission to a specific station and ending a readback with the callsign. Second, a critical transmission lost because two parties spoke on the same frequency at once; the safeguard is disciplined radio procedure, including standard phraseology and readback, so that a blocked or missed message is detected rather than assumed received.

## References

- [1] International Civil Aviation Organization, *Procedures for Air Navigation Services - Air Traffic Management (PANS-ATM)*, Doc 4444, 16th ed. Montreal: ICAO, 2016.
- [2] International Civil Aviation Organization, *Annex 11 to the Convention on International Civil Aviation - Air Traffic Services*. Montreal: ICAO.
- [3] International Civil Aviation Organization, *Manual of Radiotelephony*, Doc 9432. Montreal: ICAO.
- [4] International Civil Aviation Organization, *Annex 10 to the Convention on International Civil Aviation - Aeronautical Telecommunications, Volume II*. Montreal: ICAO.
- [5] Federal Aviation Administration, *Aeronautical Information Manual (AIM)*, current edition. Washington, DC: FAA. Pilot and controller communication and phraseology.
- [6] SKYbrary (EUROCONTROL), "Standard Phraseology" and "Non-Standard Phraseology." Available: https://skybrary.aero/articles/standard-phraseology
- [7] National Transportation Safety Board, preliminary investigation materials on the 22 March 2026 runway collision at LaGuardia Airport (Air Canada Express Flight 8646). Washington, DC: NTSB, 2026.
- [8] Congressional Research Service, "Collision at LaGuardia Airport Spotlights Long-standing Concerns Over Runway Safety," report IN12675, 2026.
- [9] K. E. Weick, "The Vulnerable System: An Analysis of the Tenerife Air Disaster," *Journal of Management*, vol. 16, no. 3, pp. 571-593, 1990.
- [10] D. Estival, C. Farris, and B. Molesworth, *Aviation English: A Lingua Franca for Pilots and Air Traffic Controllers*. London: Routledge, 2016.
- [11] Federal Aviation Administration, small UAS airspace authorization guidance, including "Airspace Guidance for Small UAS Operators" (figure source) and the Low Altitude Authorization and Notification Capability (LAANC). Washington, DC: FAA.
