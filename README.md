# Advanced State Machine for Gamemaker Studio 2

A simple system for managing two-tiered state machines.

## WHY I MADE THIS

I wanted a simple state machine that would:
  * Work every time, without running an INIT function.
  * Provide the state or sub-state without remembering a lot of functions.
  * Automatically reset the sub-state to 0 whenever we changed states.
      

##  HOW DO I INSTALL IT?

Create a new script in Gamemaker Studio 2, and paste the contents of **[ADV_STATE.txt](ADV_STATE.txt)** script into it. 

(I may come back and pretty this up later)

In the code, I've included examples of how to set up easy-to-read enums and a template state system to paste into your object's step event.


### PLEASE NOTE

State and sub-state integers must be positive whole numbers between 0 and 9999.  (No negative numbers, no decimals)



## HOW DOES IT WORK? (fast)

Imagine a state sytem with primary and sub-tier states, like this:

```      
          State_0 (Primary Tier)
              Substate_0 (Sub-tier)
              Substate_1 (Sub-tier)

          State_1 (Primary Tier)
              Substate_0 (Sub-tier)
              Substate_1 (Sub-tier)
          
          (etc)
```

Start by calling adv_state_get() (no arguments) - it returns the INT of the primary tier you're on.

Providing the current primary_tier_int returns the current sub-tier state: adv_state_get(primary_tier_int)

To move to a new primary or sub tier, call adv_state_set(primary_tier_int,[subtier_int]).


# HOW DOES THIS WORK? (documentation)
    
It uses integers stored in a local variable ("ADV_STATE") on the calling object to manage progression through each major and minor state.


## Getting the State
###  adv_state_get( [primary_tier_int] )

If no argument is provided, it returns an INT of the current primary tier state. (If no state has been set before, it sets the state to 0 and returns that)

If a primary_tier_int is provided, it returns an INT of the current sub-tier state.

### **NOTE:**
If the current primary tier is *different* than the one supplied, the system will *reset the primary state* to the supplied one and set the sub-state to zero.  If you **do not want to risk resetting the primary state**, the easiest way is to double up the call:

````
adv_state_get(adv_state_get());
````

It works this way because I wanted to make sure the system was *always* running on the state I expected it to be running on, and if something had changed, I wanted the substate reset to 0.

## Setting the State
###        adv_state_set( primary_tier_int )

###        adv_state_set( primary_tier_int/undefined, [subtier_state_int] )

If only one value is provided, it sets the current primary tier to that INT.
If two values are provided, it sets the primary tier to primary_tier_int and the sub-tier to subtier_state_int.

### Note: 
* If the first value is **undefined** (the GMS constant, not a string called "undefined"), it uses the current primary tier and only sets the sub-tier.

* If a primary tier has not been supplied or previously set, the Advanced State Machine will always set it to 0 by default.



## External State Functions

###         adv_state_external_get( object_id, [primary_tier_int] )


Functions exactly like adv_state_get, but on a separate object (provided by object_id).



###        adv_state_external_set( object_id, primary_tier_int/undefined, [subtier_state_int] )


Functions exactly like adv_state_set, but on a separate object (provided by object_id).


## Setting Up Good Enums

This isn't required: the adv_state system works fine with integers, but enums obviously make it easier to read and follow along.  Here's the format for setting up enums that I like.  (You can paste this outside the regular code in the ADV_STATE system and the enums will be declared at runtime)

````javascript

enum state { // Remember, state and substate enums must be a positive integer between 0 and 9999

	// list all the primary states first
	child,
	adult,
	elderly,
	
	// list substates in an easy-to-remember way (resetting them to 0 is also nice for easy tracking)
	
	child_baby = 0,
	child_toddler,
	child_teenager,
	
	adult_college = 0,
	adult_job,
	adult_parent,


	elderly_retired = 0,
	elderly_grandparent,
	elderly_motorcycleHooligan

}



````

