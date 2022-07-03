# Advanced State Machine for Gamemaker Studio 2

A simple system for managing two-tiered state machines.

This has since expanded to include three simple systems for managing basic state machines:

1) [The state machine manager functions](#the-state-system).
2) [The processor check system](#advanced-processor-system) (for performing tasks every X steps)
3) [The timer system](#advanced-timer-system)

## If it's so simple, why do I call it **"advanced"**?  

BRANDING, KIDS.

BRANDING.

(I honestly don't know)

## WHY I MADE THIS

I wanted a simple state machine that would:
  * Work every time, without running an INIT function.
  * Provide the state or sub-state without remembering a lot of functions.
  * Automatically reset the sub-state to 0 whenever we changed states.

As time went on, I found it useful to add the ability to set a timer, change a state, and then have the state system wait until the timer elapsed (or had partially elapsed) before changing states again.

## HOW DO I INSTALL IT?

1) Download the adv_state_v1.3.yymps file

2) In Gamemaker Studio 2, click Tools -> Import Local Package

3) Navigate to the downloaded file, and click "Add All."

In the code, I've included examples of how to set up easy-to-read enums and a template state system to paste into your object's step event.

If you want to use the advanced processor system (for checking things every X steps), you'll need to [add one function call somewhere in a Step_Begin event](#how-do-i-set-up-the-advanced-processor-system).

# THE STATE SYSTEM
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

### PLEASE NOTE

State and sub-state integers must be positive whole numbers between 0 and 9999.  (No negative numbers, no decimals)



## HOW DOES THIS WORK? (documentation)
    
The state system uses integers stored in local variables ("ADV_STATE" and "ADV_STATE_LAST") on the calling object to manage progression through each primary tier and sub-tier state.

These functions are designed to be run on the object whose state you want to track.  To get/set states on other objects, check the [External functions](#external-state-functions) below.

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


## Retrieving the previous state

###        adv_state_last_state_get()

###        adv_state_last_substate_get()

Returns the previous state / substate to the one we're on now.  If there was no previous substate, it returns -1.

### Note:
* This is the only occasion where a separate function exists to get the substate.



## External State Functions

###         adv_state_external_get( instance_id, [primary_tier_int] )


Functions exactly like adv_state_get, but on a separate object (provided by instance_id).



###        adv_state_external_set( instance_id, primary_tier_int/undefined, [subtier_state_int] )


Functions exactly like adv_state_set, but on a separate object (provided by instance_id).


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
# Advanced Processor System

This is still a work in progress, but the fundamentals are this: using the function as described, you can check if you are on step X, and if so, perform functions as you see fit.

## How do I set up the Advanced Processor System?

In the Step_Begin event of a Singleton or manager object (something permanent), add this function call:
```gms
adv_processor_tick()
```

That's all.

## How do I use the Advanced Processor System?

The Advanced Processor System runs on a tick cycle of 60 steps. (You can change this by modifying the **__ADV_STATE_TICK_MAXIMUM** macro.

* [Functions](#processor-functions)
* [Examples](#advanced-processor-examples)

## Processor Functions
### adv_processor_check(_int) 

Returns true if we are on _int frame within the Advanced Processor tick cycle.

I find this useful for spreading intense processing over several frames.

### adv_processor_check_every(_int)

Returns true every _int frame within the Advanced Processor tick cycle.

If you're going to use this function, I recommend setting the **__ADV_STATE_TICK_MAXIMUM** macro to be the same as your game's framerate.  

If you supply 4 (assuming 60 ticks maximum), it will run 15 times per cycle.


### Advanced Processor Examples

#### Using Adv_Processor_Check()

```gms

#macro __ADV_STATE_TICK_MAXIMUM		60

if(adv_processor_check(3)) // Returns "true" once every second, on the 4th tick.
	{
	show_debug_message("We're on the 4th tick.");
	
	run_processor_intensive_function_part1();
	}

if(adv_processor_check(7)) // Returns true once per second, on the 8th tick
	{
		run_processor_intensive_function_part2();
	}

```

#### Using Adv_Processor_Check_Every()

These calculations assume a 60 frames per second game.
```gms

#macro __ADV_STATE_TICK_MAXIMUM		60 

if(adv_processor_check_every(3)) // Returns "true" twenty times per second, on every 3rd tick.
	{
		check_if_players_are_still_connected();
	}

if(adv_processor_check(18)) // Returns true 4 times per second, on 0, 18, 36, and 54
	{
		build_tax_liability_alibi();
	}

```


# Advanced Timer System

## How do I use it?
A simple timer system can make managing states a lot easier.  I use this a lot when waiting to trigger complicated animation sequences.  "I know it should take roughly X seconds to finish before I get to Y, so I will set this timer, and once it's up, I jump to the next state."

Also useful for delaying back-end changes until the GUI has caught up (inflicting damage, killing the player, etc)

It isn't a "trigger" unto itself: think of it more like a stopwatch.  Set it once, and then you run a separate check to see if it's finished, and if so, you move to the next state.

It is essentially a timestamp, so it requires zero processor overhead to run.

Please note: this function group works by setting a local timestamp on an object, and as such, should be run on the object doing the checks (similar to [the state system](#the-state-system)).  You can set and check the timer from other objects using the [External function set](#external-timer-functions). 

* [Functions](#timer-functions)
* [Examples](#timer-examples)


## Timer Functions
### adv_timer_set(_duration_in_seconds)
Sets the timer for a given number of seconds.  Nothing will trigger after _duration_in_seconds - it's just a simple way of keeping track of when it happened.

### adv_timer_elapsed([time_in_seconds])

Returns **true** if the timer has elapsed.

If you provide [time_in_seconds], it will tell you if that many seconds have elapsed since the timer was set.


## External Timer Functions

### adv_timer_external_set(_instance_id,_duration_in_seconds)
Identical to the adv_timer_set(), but it sets the timer on a different object than the calling one. (Useful if you want to call the function from a struct)


### adv_timer_external_elapsed(_instance_id,[_time])
Same as adv_timer_elapsed() (Including the optional "time" argument), but checks the timer on a different object than the calling object.

## Timer Examples

Below is an example of how I'll use timers to help manage how I move through states in my objects.

```gms

// Let's assume we're in state.first_state_adult state, and the state.first_substate_love substate

case state.first_substate_love:
	{
		start_my_four_second_heart_animation(); 
	
		// Setting a timer for 4 seconds
		adv_timer_set(4); 	
		
		// We want to stay in the current state, so we supply nothing to the first argument
		// But we want to move to the next substate
		
		adv_state_set(,state.first_substate_marriage);	
		bury_player_in_legal_documents();
		
		break;
	}

case state.first_substate_marriage:	
	{
		// Once we arrive in this state, check if the timer is up
		if(adv_timer_elapsed())
			{
				// Our animation has finished playing, let's do our back-end cleanup and move to next state
				set_pregnant(true);
				set_breathe(breathtype.cheetah_attacking_me);
				
				adv_timer_set(567648000); // Set timer for 18 years
				adv_state_set(,state.first_substate_parent);
				
			}
		break;
	}

case state.first_substate_parent:
	{
		pay_money(money.all);
		set_rest(-1);
		
		if(adv_timer_elapsed())
			{
				set_breathe(breathetype.freedom);
				
				adv_state_set(state.second_state_golden_years);
			}
	
		break;
	}

```
