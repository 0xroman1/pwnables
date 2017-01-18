Well this is a game that allows us to play blackjack. Either by looking through the source code, or by actually playing it we can tell that it asks us how to play, make a bet, and then we play. Now looking through the source code (and just being particularly curious about the betting function) I noticed something about the betting code.



```
nt betting() //Asks user amount to bet
{
 printf("\n\nEnter Bet: $");
 scanf("%d", &bet);
 
 if (bet > cash) //If player tries to bet more money than player has
 {
        printf("\nYou cannot bet more money than you have.");
        printf("\nEnter Bet: ");
        scanf("%d", &bet);
        return bet;
 }
```



It checks to see if the bet is more than what the player has, but not if he has what is less. -$1000000 is less than $5. If you loose with a bit of -$1000000, then you will essentially earn $1000000 dollars. So by placing that bet (or an even lower bet) and losing, the next time you play it will output the flag. This exploit could be patched in the code by checking to see if the bet is less than 0.


Flag: YaY_I_AM_A_MILLIONARE_LOL
