2026-10-21 15:18
Status: #baby 
Tags: [[maths]]
# Quadratic standard form to vertex form

You may want to know what the vertex of a parabola is from a standard form of a quadratic equation. Lets say builder throws a brick off a platform and that can be modelled with this function. Were the function is in metres and $t$ is in seconds.
$$f(t)=-5t^2+20t+105$$
Your task is to change this into vertex form so you can pull out the vertex of a parabola. Lets break it down. 

Step 1 you need to factor out the $-5$ in front of the $t^2$ because we are going to be using completing the square method, this only works when there is no coefficent in front of square term.
$$f(t)=-5(t^2-4t)+105$$
Be careful of the sign changed when factoring. 

Why not set it to zero and divide by $-5$? 
Well because that's not what we are asked to do, that would be getting the x-intercepts not the vertex.

Why don't you factor out a $-5t$ instead of just $-5$?
This would NOT allow us to complete the square and we would arrive at a dead end.

Why only factor out $-5$ from part of the equation?
This speeds up the computation and reduces steps needed to completing the square and turn things into a vertex form. There is nothing wrong with factoring out a $-5$ it just makes this more difficult to work with for example $f(t)=-5t^2+20t+101$ now we have a fraction to work with and everything gets more annoying.

Step 2 Setting up for completing the square inside the brackets we do this by
- Looking the middle term $-4t$ 
- Divide the coeffiecent by 2 so $-4 \div 2 = -2$ 
- Then square it $(-2)^2 = 4$ 
When the add a $+4$ to the equation inside the brackets like this
$$f(t)=-5(t^2-4t + 4)+105$$
So now it's possible to complete the square.

Step 3 Fixing the equation and completing the square.
#### WARNING
We are not done yet because the equation so it's no longer true. We need to substract the same thing we added to make the equation make it true again.
#### DANGER
Here is very easy to make this mistake, you see that $+4$ and you simply think "I will just substract 4" writing $$f(t)=-5(t^2-4t + 4)+105 - 4$$
This is **WRONG** you need to realise that the $+4$ is within the brackets of a of $-5()$ so if you want to substract the same thing you need to add $20$

Why? Lets break it down I will substracting $4$ within the brackets for a moment and also complete the square.
  $$f(t)=-5(t^2-4t + 4 - 4)+105$$
Complete the square
  $$f(t)=-5((t - 2)^2 - 4)+105$$
Then we distribute the $-5$
$$f(t)=-5(t - 2)^2 + 20 + 105$$
Simply
$$f(t)=-5(t - 2)^2 + 125$$

This is finished but if you wanted to skip a step you can notice that the coefficient is $-5$ and you added $4$ so the thing you need to substraction is $-4 \times -5$ which is $+20$ 
  $$f(t)=-5(t^2-4t + 4)+ 20 + 105$$
Complete the square and simplify
  $$f(t)=-5(t - 2)^2+125$$
Step 4 Getting the vetex of the parabola

Now we can easily get the vertex of the parabola from $f(t)$ by picking a $t$ that make $-5(t-2)^2 = 0$ which will leave us with the maxima of the parabola for a given $t$ which is $125$.

Answer: At 2 seconds the brick is at the highest point at 125 metres above the ground.
# References
##### Main Notes
#### Source Notes
