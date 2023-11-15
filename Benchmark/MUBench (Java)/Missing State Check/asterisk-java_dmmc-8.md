## Repository

URL: https://github.com/asterisk-java/asterisk-java.git

Commit: 304421c261da68df03ad2fb96683241c8df12c0a^1

## API

java.util.List

## Caller

```java
package org.asteriskjava.manager.internal;

class EventBuilderImpl extends AbstractBuilder implements EventBuilder
{
    private static final Set<String> ignoredAttributes = new HashSet<String>(Arrays.asList("event"));
    private Map<String, Class<?>> registeredEventClasses;

	public ManagerEvent buildEvent(Object source, Map<String, Object> attributes)
    {
        ManagerEvent event;
        String eventType = null;
        Class<?> eventClass;
        Constructor<?> constructor;

        if (attributes.get("event") == null)
        {
            logger.error("No event type in properties");
            return null;
        }

        if (attributes.get("event") instanceof List ) 
		{
            List<?> eventNames = (List<?>) attributes.get( "event" );
            if (eventNames.size() > 0 && "PeerEntry".equals(eventNames.get(0))) 
			{
                // List of PeerEntry events was received (AJ-329)
                // Convert map of lists to list of maps - one map for each PeerEntry event
                int peersAmount = attributes.get("listitems") != null ?
                    Integer.valueOf((String) attributes.get("listitems")) :
                        eventNames.size() - 1; // Last event is PeerlistComplete
                List<Map<String, Object>> peersAttributes = new ArrayList<Map<String, Object>>();
                for (Map.Entry<String, Object> attribute : attributes.entrySet()) 
				{
                    String key = attribute.getKey();
                    Object value = attribute.getValue();
                    for (int i = 0; i < peersAmount; i++) 
					{
                        Map<String, Object> peerAttrs;
                        if (peersAttributes.size() > i) 
						{
                            peerAttrs = peersAttributes.get(i);
                        } 
						else 
						{
                            peerAttrs = new HashMap<String, Object>();
                            peersAttributes.add(i, peerAttrs);
                        }
                        if (value instanceof List) 
						{
                            peerAttrs.put(key, ((List<?>) value).get(i));
                        } 
						else if (value instanceof String && !"listitems".equals(key)) 
						{
                            peerAttrs.put(key, value);
                        }
                    }
                }
                attributes.put("peersAttributes", peersAttributes);
                eventType = "peers";
            }
        } 
		else 
		{
            if (!(attributes.get("event") instanceof String)) 
			{
                logger.error("Event type is not a String or List");
                return null;
            }

            eventType = ((String) attributes.get("event")).toLowerCase(Locale.US);

            // Change in Asterisk 1.4 where the name of the UserEvent is sent as property instead
            // of the event name (AJ-48)
            if ("userevent".equals(eventType))
            {
                String userEventType;

                if (attributes.get("userevent") == null)
                {
                    logger.error("No user event type in properties");
                    return null;
                }
                if (!(attributes.get("userevent") instanceof String))
                {
                    logger.error("User event type is not a String");
                    return null;
                }

                userEventType = ((String) attributes.get("userevent")).toLowerCase(Locale.US);
                eventType = eventType + userEventType;
            }
        }

        eventClass = registeredEventClasses.get(eventType);
        if (eventClass == null)
        {
            logger.info("No event class registered for event type '" + eventType + "', attributes: " + attributes
                    + ". Please report at https://github.com/asterisk-java/asterisk-java/issues");
            return null;
        }

        try
        {
            constructor = eventClass.getConstructor(new Class[]{Object.class});
        }
        catch (NoSuchMethodException ex)
        {
            logger.error("Unable to get constructor of " + eventClass.getName(), ex);
            return null;
        }

        try
        {
            event = (ManagerEvent) constructor.newInstance(source);
        }
        catch (Exception ex)
        {
            logger.error("Unable to create new instance of " + eventClass.getName(), ex);
            return null;
        }

        if (attributes.get("peersAttributes") != null && attributes.get( "peersAttributes" ) instanceof List) {
            // Fill Peers event with list of PeerEntry events (AJ-329)
            PeersEvent peersEvent = (PeersEvent) event;
            // TODO: This cast is very ugly, we should review how attributes are being passed around. 
            for( Map<String, Object> peerAttrs : (List<Map<String, Object>>) attributes.get("peersAttributes")) {
                PeerEntryEvent peerEntryEvent = new PeerEntryEvent( source );
                setAttributes(peerEntryEvent, peerAttrs, ignoredAttributes);
                List<PeerEntryEvent> peerEntryEvents = peersEvent.getChildEvents();
                if (peerEntryEvents == null) 
				{
                    peerEntryEvents = new ArrayList<PeerEntryEvent>();
                    peersEvent.setChildEvents(peerEntryEvents);
                }
                peerEntryEvents.add(peerEntryEvent);
            }
            peersEvent.setActionId((peersEvent.getChildEvents().get(0).getActionId()));
        } 
		else 
		{
            setAttributes(event, attributes, ignoredAttributes);
        }

        // ResponseEvents are sent in response to a ManagerAction if the
        // response contains lots of data. They include the actionId of
        // the corresponding ManagerAction.
        if (event instanceof ResponseEvent)
        {
            ResponseEvent responseEvent;
            String actionId;

            responseEvent = (ResponseEvent) event;
            actionId = responseEvent.getActionId();
            if (actionId != null)
            {
                responseEvent.setActionId(ManagerUtil.stripInternalActionId(actionId));
                responseEvent.setInternalActionId(ManagerUtil.getInternalActionId(actionId));
            }
        }

        return event;
    }
}

```

## Description

In line 323 the `value` is cast to `List` and its `i`s element is accessed
without prior check that such an element exists.


## Rule

Missing State Check

## Response

### Confidence

85

### Types

Missing Null Check

### Crash

True

### Snippet

```java
if (peersAttributes.size() > i) 
						{
                            peerAttrs = peersAttributes.get(i);
                        } 
						else 
						{
                            peerAttrs = new HashMap<String, Object>();
                            peersAttributes.add(i, peerAttrs);
                        }
```

### Fix

```java
peerAttrs = peersAttributes.get(i);
if(peerAttrs == null) {
     peerAttrs = new HashMap<String, Object>();
     peersAttributes.add(i, peerAttrs);
}
```

### Description

Instead of checking the index of the list, which can lead to an IndexOutOfBoundsException if the index is out of the list's range, we should get the element at the specified index and check if it's null before adding a new HashMap to the list.

## Review

Found the expected misuse? **False**

