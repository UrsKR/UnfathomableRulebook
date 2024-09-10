---
layout: page
title: Dynamic Unfathomable rulebook
toc: true
toc_stop_autofire: true
---

<script type="text/javascript">

function toggleCL() {
  return toggle('#cylonleader');
}

function toggle(id) {
  if (readCheckbox(id)) { 
    $(id).prop('checked', false);
  } else { 
    $(id).prop('checked', true);
  }
  flipSwitches();
  return false; 

}

function readCheckbox(id) {
  return $(id).is(':checked')
}

function enable(id) {
  $(id).removeAttr('disabled');
}

function forbidCheckbox(id) {
  $(id).prop('checked', false)
       .prop('disabled', true);
}

function mandateCheckbox(id) {
  $(id).prop('checked', true)
       .prop('disabled', true);
}


function forbidMenu(id) {
  $(id).prop('disabled', true);
  if ( $(id).is(':selected')) {
    $(id).removeAttr('selected');
  }
}

function validateForm() {
  enable('#players3');
  enable('#players4');
  enable('#players5');
  enable('#players6');	
  if (readCheckbox('#abyss')) {
    forbidCheckbox('#firstgame');
  } else {
    enable('#firstgame');
  }
  if (readCheckbox('#firstgame')) {
    forbidCheckbox('#abyss');
  } else {
    enable('#abyss');
  }
}

function highlight(theClass) {
  // Don't highlight the "no" classes, except for "nosympathizer"
  if (theClass === "nosympathizer" || ! /^no/.test(theClass)) {
    $('.' + theClass).css({"background-color":"lightyellow"});
  }
}

function unhighlight(theClass) {
  $('.' + theClass).css({"background-color":""});
}

function flipSwitches () {
  // Step 1: validate the form. Uncheck and disable items that aren't
  // allowed.
  
  validateForm();
  
  // Step 2: Collect lists of classes to hide and show.
  var showThese = [];
  var hideThese = 
  var pullFrom = 'input,option';
  showThese.push('basics');
  showThese.push('setup');
  $(pullFrom).each(function(index, element) {
    if ($(this).is(':checked')) {
      showThese.push($(this).attr('id'));
      hideThese.push('no'+$(this).attr('id'));
    } else {
      showThese.push('no'+$(this).attr('id'));
      hideThese.push($(this).attr('id'));
    }
  });  
  
  if (readCheckbox('#abyss')) {
    showThese.push('expansion');
    hideThese.push('noexpansion');
  } else {
    showThese.push('noexpansion');
    hideThese.push('expansion');
  } 
  if (readCheckbox('#abyss')) {
    showThese.push('boons');
    hideThese.push('noboons');
  } else {
    showThese.push('noboons');
    hideThese.push('boons');
  }
  if (readCheckbox('#players3')) {
    showThese.push('players3');
    hideThese.push('players4');
    hideThese.push('players5');
    hideThese.push('players6');
  } 
  if (readCheckbox('#players4')) {
    hideThese.push('players3');
    showThese.push('players4');
    hideThese.push('players5');
    hideThese.push('players6');
  } 
  if (readCheckbox('#players5')) {
    hideThese.push('players3');
    hideThese.push('players4');
    showThese.push('players5');
    hideThese.push('players6');
  } 
  if (readCheckbox('#players6')) {
    hideThese.push('players3');
    hideThese.push('players4');
    hideThese.push('players5');
    showThese.push('players6');
  } 
	
  // Step 3: Show all the classes that need showing. 
  for (i in showThese) {
    $('.'+showThese[i]).show();
    // Highlight if requested
    if (readCheckbox('#highlight')) {
      highlight(showThese[i]);
    } else {
      unhighlight(showThese[i]);
    }
  }
  // Step 4: Hide all the classes that need hiding. Since we do this 
  // last, that means a given tag needs *all* elements to be visible,
  // or in other words, each list of tags is ANDed together.
  for (i in hideThese) {
    $('.'+hideThese[i]).hide();
  }
  
  // Step 5: Fix the rowspan on the basestar attack table. It has to
  // change based on the options set.
  var rowspan = 3;
  if (readCheckbox('#daybreak')) {
    // Additional one for assault raptors
    rowspan++;
  }
  if ( readCheckbox('#cylonfleet')) {
    // Remove the nuke row
    rowspan--;
  }
  $('#basestardamage').attr('rowspan', rowspan);
    
  // Step 5: Refresh the table of contents.
  $('#toc').toc({showSpeed: 0});
  
  // Save to local storage
  save();
  
  // Update the share URL box
  var url = window.location.origin + window.location.pathname + "?" + buildStateString();
  $('#generatedUrl').val(url);

}

function save() {
  if (window.sessionStorage){
    try {
      $('input,option').each(function(index, element) {
        if (readCheckbox('#'+$(this).attr('id') )) { 
          window.sessionStorage.setItem($(this).attr('id'), "1");
        } else {
          window.sessionStorage.removeItem($(this).attr('id'));
        }
      });  
    } catch (err) {
      // Probably not allowed. That's okay, this
      // feature is optional so silently failing
      // is okay. 
    }
  }
}

// find all the selected / checked items and return a
// querystring representing them
function buildStateString() {
  qs = [];
  $('input,option').each(function(index, element) {
    id = $(this).attr('id');
    if (readCheckbox('#' + id)) {
      qs.push(id);
    }
  });
  return qs.join('&');
}

// enable this id (check it or select it)
function setValue(id) {
  if (!/^[a-zA-Z][a-zA-Z0-9\-\_]+$/.test(id)) {
    return false;
  }
  var el = $('#'+id);
  
  if (el.length === 0) {
    return false;
  }
  
  if (el.is('option') || el.is('input')) {
    el.prop('checked', true);
    el.prop('selected', true);

    return true;
  }
  
  return false;
}

// This is the page initialization code
$(function () {
  // Obviously, we have JavaScript if this is running.
  $(".nojs").hide();
  $(".js").show();

  var foundConfig = false;
  // queryparam exists?
  var qs = window.location.search;
  if (!!qs) {
    // use querystring to set values
    qs = qs.replace("?", '').split('&');
    for (var i=0; i < qs.length; i++) {
      if (setValue(qs[i])) {
        foundConfig = true;
      }
    }
  }
  
  if (foundConfig) {
    // Disable configuration, since this is preconfigured.
    // But they can choose to remove the configuration if desired.
    $(".preconfigured").show();
    $(".nopreconfigured").hide();
  } else {
    // state exists?
    if (window.sessionStorage){
      for (id in window.sessionStorage) {
        setValue(id);
      }
    }
    // Show the real config form
    $("#configform").show();
    // There is no preconfiguration here. Set CSS accordingly.
    $(".preconfigured").hide();
    $(".nopreconfigured").show();

  }
  $('#configform').change(flipSwitches);
  flipSwitches();
});

</script>

<form id="configform" style="display: none;">
  <fieldset id="configbox">
    <legend>Configuration:</legend>
    <label><input type="checkbox" name="abyss" id="abyss"> From The Abyss</label><br>
    <label>Player Count:
      <select id="playercount">
        <option value="3" id="players3" selected>3</option>
        <option value="4" id="players4">4</option>
        <option value="5" id="players5">5</option>
        <option value="6" id="players6">6</option>
      </select>
    </label>
    <hr>
    <label><input type="checkbox" name="firstgame" id="firstgame">Variant: First Game limitations</label><br>
    <hr>
    <label>Share this configuration: 
      <input style="width: 100%;" type="text" id="generatedUrl" name="generatedUrl" />
    </label>
  </fieldset>
</form>

<form id="preconfigform" class="preconfigured" style="display: none;">
  <fieldset id="preconfigbox">
    <legend>Configuration:</legend>
    <p>This link was pre-configured. <a href="{{ site.baseurl}}rulebook.html">
    Click here to go back to the configurable rulebook.</a></p>

    <p>
    This configuration includes:</p>
    <ul>
      <li class="abyss"> From the Abyss
        <ul> 
          <li>From the Abyss Expansion</li>
      <li>Player Count:
        <ul>
          <li class="players3">3</li>
          <li class="players4">4</li>
          <li class="players5">5</li>
          <li class="players6">6</li>
        </ul>
      </li>
    </ul> 
  </fieldset>
</form>

<div class="basics" markdown="1">
## Introduction

This rulebook is intended to be a single resource for the official rules of [Unfathomable by Fantasy Flight Games](https://www.fantasyflightgames.com/en/unfathomable/) and its expansion. The goals are for it to be complete and unambiguous, incorporating the published rules included with the games as well as clarifications and rulings made later, so that no one has to dig through rulebooks, reference, errata, official FAQs and other resources.

For beginning players, it's probably best to use the official base game rulebook to learn the game. This rulebook has lots of detail, even when all the expansions are turned off, which is probably going to hurt more than help. But if you are playing the game and have a question that the official rulebook doesn't seem to answer, take a look. An experienced player should be able to use this to teach new players the game, since they can explain the basics themselves and know what details can be ignored to start. 

Configure which rules are being included by using the form at the top of the page. Some rules change based what is included, and the rulebook will change as you configure it. 

By default, this rulebook sticks to the official rules and rulings by Fantasy Flight Games.

## The Setting

The year is 1913.  
You are the passengers and crew aboard the steamship SS Atlantica, a small passenger ship operated by Fairmont Shipping Co. that is traveling across the Atlantic Ocean en route to Boston, Massachusetts. During the first days of the voyage, there are rumors that the lookouts have spotted large, dark shapes in the water following behind the ship. Some of the ship’s occupants have been behaving strangely, making croaking sounds or staring unblinkingly at the open sea. You, too, are not feeling quite yourself, your nights plagued by dreams of eerie underwater landscapes teeming with shadowy figures. Tensions are high, nerves are frayed, and the ship is a powder keg ready
to explode.  

On the third night of the seven-day voyage, a passenger is found murdered in the ship’s chapel. Her body is slumped over an ancient tome with ritual components laid out around her on the chapel’s altar. An illustration on the open page of the tome depicts a figure in the center of a shimmering circle. Outside of the circle is a swarm of fleeing monsters. You do not have long to contemplate the meaning of the image before
cries and screams of horror go up around the ship. Something has emerged from the water and is climbing aboard!

## Overview
Unfathomable is a game of hidden loyalties, intrigue, and paranoia for three to six players. Some players are humans who are
fighting for the survival of the ship, its passengers, and its crew. But some players are traitors, sent aboard the SS Atlantica by the Deep
Ones to ensure the ship never reaches port! Because player loyalties are hidden, determining who is friend and who is foe is critical to
winning in Unfathomable. 

During the game, the human players arm themselves with items, fight Deep Ones, rescue passengers, and make sure that the ship stays afloat. At the end of every human player’s turn is a crisis that either poses a difficult decision for one player or challenges all players to work together to overcome the crisis. If the humans can work together successfully, the ship will eventually reach port, resulting in a win for the humans.  

However, the traitors are hiding among the humans, secretly sabotaging the ship and doing all they can to ensure its doom. If the ship sinks before it can reach its final destination, the traitors win.
</div>

<div class="setup" markdown="1">
## Game setup

1. **Game Board**: Place the game board in the center of the table.
2. **Set Tracks**: Place the travel and ritual track tokens on the Start space of their respective tracks.
3. **Set Dials**: Set each resource dial to its starting value of 8.
4. **Create Supply**: Place the Deep One figures, passenger tokens, traitor rings, and eight-sided die next to the board. Flip all passenger tokens facedown and mix them thoroughly.
5. **Create Skill Decks**: Separate the skill cards by type (influence, lore, observation, strength, will, and treachery), shuffle each type into its own deck, and place each deck face down beside the game board next to the corresponding label.
6. **Create Damage and Mythos Decks**: Shuffle the damage deck and place it beside the game board next to its label. Shuffle the mythos deck and place it near the board.
7. **Place Monsters and Passengers**: Place Father Dagon and Mother Hydra in the Deep. Place six additional Deep Ones and two passenger tokens as follows: 2 Deep Ones in each of sea spaces 1 and 2, 1 Deep Ones in each of deck spaces 1 and 2 and 1 passenger token in each of deck spaces 3 and 4.
8. **Create Chaos Deck**: Take two cards from each skill deck (except treachery) and, without looking at them, shuffle them together to create the chaos deck. Place the deck in the chaos deck space on the board.
9. **Select Characters**: Randomly select a player to be the current player and give them the current player token. Then, starting with the current player and proceeding clockwise, each player chooses one character to play and takes that character’s sheet. The “Suggested Characters” sidebar has a list of recommended characters to use for your first game according to the number of players. Return the remaining character sheets to the game box.
10. **Create Play Areas**: Each player takes the feat card and standee that matches their chosen character as well as a double-sided player
reference sheet and places them in their play area. Return the remaining feat cards and standees to the game box.
11. **Gather Items**: Each player takes the starting item listed on the back of their character sheet from the item deck and places it faceup in their play area. Shuffle the remaining items to create the item deck and place it beside the game board next to its label.
12. **Place Characters**: Each player places the standee for their chosen character in the starting space on the board listed on the back of their character sheet.
13. **Draw Skill Cards**: Each player except the current player draws the five skill cards listed at the bottom of their chosen character’s sheet. **The current player does not start the game with any cards in hand.**
14. **Create Waypoint Deck**: Shuffle the waypoint cards and give the deck and the Captain title card to the player with the
character who is highest on the Captain Line of Succession list (provided on the 14 back of the Captain title card).
15. **Create Spell Deck**: Shuffle the spell cards and give the deck and the Keeper of the Tome title card to the player with the character who is highest on the Keeper Line of Succession list (provided on the back of the Keeper of the Tome title card).
16. **Create Loyalty Deck**: Create the loyalty deck by combining the loyalty cards listed below:
    <ul class="players3">
	<li>1 Hybrid Loyalty Card</li>
        <li>5 Human Loyalty Cards</li>
    </ul>
    <ul class="players4">
	<li>1 Hybrid Loyalty Card</li>
        <li>7 Human Loyalty Cards</li>
    </ul>
    <ul class="players5">
	<li>2 Hybrid Loyalty Card</li>
        <li>8 Human Loyalty Cards</li>
    </ul>
    <ul class="players6">
	<li>2 Hybrid Loyalty Card</li>
        <li>10 Human Loyalty Cards</li>
    </ul>
</div>
