2026-04-09 07:24
Status: #baby 
Tags: [[C++]]
# How dymanic_cast actually works

The downcasting is the issue when it comes to RTTI. Look at the comments for what works and what doesn't everything here compiles

```c++
class Colour
{
public:
	Colour(char character) : characterofcolour(character)
	{
	}

	virtual void printColour()
	{
		std::cout << "This function doesn't colour" << std::endl;
	}
protected:
	char characterofcolour;
};

class Red final: public Colour
{
public:
	Red(char character) : Colour(character)
	{
	}
	virtual ~Red() = default;

	virtual void printColour() override
	{
		std::cout << "This is the colour red" << std::endl;
		std::cout << "The caracter value :" << characterofcolour << std::endl;
	}

	void print1()
	{
		std::cout << "Wow!" << std::endl;
	}
};

class Blue final : public Colour
{
public:
	Blue(char character) : Colour(character)
	{
	}
	virtual ~Blue() = default;

	virtual void printColour() override
	{
		std::cout << "This is the colour blue" << std::endl;
		std::cout << "The caracter value :" << characterofcolour << std::endl;
	}
};

int main() 
{
	//This downcasting works
	Colour* colourBlue = new Blue{'B'};

	colourBlue->printColour();
	Blue* blue = dynamic_cast<Blue*>(colourBlue);
	if (blue)
	{
		blue->printColour();
	}

	//This does not work! Pay attention to the new Colour constructor.
	Colour* colourBlueTwo = new Colour{ 'N' };

	colourBlueTwo->printColour();
	Blue* blueTwo = dynamic_cast<Blue*>(colourBlueTwo);
	if (blueTwo)
	{
		blue->printColour();
	}

	//This downcasting works even though the classes are different.
	Colour* colourRed = new Red{ 'R' };

	colourRed->printColour();
	Red* red = dynamic_cast<Red*>(colourRed);
	if (red)
	{
		red->printColour();
	}

	return 0;
}
```

The reason you would want to have does this ever is because you can have something like an array of class pointers in a flat arrary that are all the parents of children. Then you just emplace back or whatever, the children class and everything works out.
# References
##### Main Notes
[[What is RTTI]]
#### Source Notes
[[RTTI (Run-Time Type Information) in C++]]