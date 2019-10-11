### Blog Post

Blog Post on IEnumerables vs Lists and Arrays ( title TBD )

My name is Ben Muller and I am an aspiring software engineer. I work at MYOB in Sydney, Australia and I have been in the [Future Makers Academy](https://www.myob.com/au/careers/graduate-program) (“FMA”) for just over two months. As part of the FMA protégés have the incredible opportunity to present their code at milestones referred to as quorum reviews (“QR”). At the QR a protégé presents a coding challenge and walks through their solution, business logic, design choices and overall understanding of C# and the language design principles associated with it. The QR is done in front of six senior developers, a principal developer ( essentially one of the top developers at the company ) and the FMA coordinator.

The QR went well, I was definitely a little bit nervous at first but after a few minutes I realised that everyone in the room was here to help me grow and learn, not to shoot me down for anything I had done wrong. It also dawned upon me that this is as close to a once in a lifetime opportunity as they come, a chance to learn from the devs in the room and see all of their opinions on how I can better my craft. I often find myself seeking out others opinions on my code to see if I am doing things in the “correct” way, and here I am with the vehicle to do that with eight different opinions.

At work I use [Jetbrains Rider IDE](https://www.jetbrains.com/rider/) for all of my C# development, it is incredibly powerful and user friendly once you get the hang of it. It is so powerful that it can often recommend certain types of refactorings to make to your code. Starting out as a developer this can often be a trap. You can find yourself accepting Riders refactoring suggestions and not understanding why. I was slightly guilty of this. In my QR I received some feedback to reinforce my understanding of Collections and in particular when to use each one and the differences between them eg when to use IEnumerable vs List vs Array. 

After the QR I set out to up skill in the topic and gain a much deeper understanding. I found that there was a lack of straight forward up to date information on the topic, so I decided to write a post about it for future aspiring developers in my position.

So, when I started looking into when to use  IEnumerable vs List vs Array this was one of the first things I came across in a SO post.


``IEnumerable describes behavior, while List is an implementation of that behavior. When you use IEnumerable, you give the compiler a chance to defer work until later, possibly optimizing along the way. If you use ToList() you force the compiler to reify the results right away.``

At first, this made a bit of sense to me, but I didn’t really understand the statement about:

“When you use IEnumerable, you give the compiler a chance to defer work until later, possibly optimizing along the way.”

So let’s explore this a little more with a bit of code…

```
static void Main()
{
  var names = new List<string> {"sam", "alexia", "simon", "sumanth", "tony", "sam", "amr", "mark", "drew"};
  
  var moreThanFourLetters = names.Where(w => w.Length > 4);
  
  names[0] = "benjamin";

  foreach (var name in moreThanFiveLetters)
  {
      Console.WriteLine(name);
  }
}
```

In the above code we have some sample names in a List.

Next we have a LINQ query to find the words that have more than 5 letters. Then we reassign the first word in the names list to benjamin.

Now, what would you expect this code to write to the console?

I know that at first I thought the program would output `alexia, sumanth`.

Then I thought further about the SO post.

“When you use IEnumerable, you give the compiler a chance to defer work until later, possibly optimizing along the way.”

defer work until later, hey?

So what is actually happening here? 
When you run a LINQ it returns an IEnumerable\<T>, so it doesn't actually save the results of the query into memory. It just saves the actual query and delays the execution until later when it is looped over or a value type is extracted from the variable it is saved in.

So in this specific example when the code is iterating over the `moreThanFiveLetters` IEnumerable it executes the query on the `names` List and its current state at the time of the loops execution.

This means that the above code actually writes `benjamin, alexia, sumanth` to the console. 

To really drum home the concept let's consider the following code ( it is the same as the above code, only I have turned the IEnumerable into a List with .ToList() ):

```
var names = new List<string> {"sam", "alexia", "simon", "sumanth", "tony", "sam", "amr", "mark", "drew"};
            
var moreThanFiveLetters = names.Where(w => w.Length > 5).ToList();

names[0] = "benjamin";

foreach (var name in moreThanFiveLetters)
{
    Console.WriteLine(name);
}
```

In the above example what do you think is written to the console?

If you guessed `alexia, sumanth` then you are correct!

In this example casting the IEnumerable via `.ToList()` means that the `moreThanFiveLetters` variable is now a List which is a reference type and `moreThanFiveLetters` is an instance of the List type.

That is a bit of a mouthful. Think of it like this, List is a class and it has a bunch of properties and methods associated to that class and therefor available to `moreThanFiveLetters` when we cast it to a list. 

Because `moreThanFiveLetters` is now an instance of a List and its own object, when we execute:

```
names[0] = "benjamin";
```
`moreThanFiveLetters` technically doesn't know about names  and therefor is unaffected.

So when we iterate over `moreThanFiveLetters`, `alexia, sumanth` is written to the console.

Key Findings:
- IEnumerable is read-only and List is not.
- IEnumerable types have a method to get the next item in the collection. It doesn't need the whole collection to be in memory and doesn't know how many items are in it, foreach just keeps getting the next item until it runs out.
- List implements IEnumerable, but represents the entire collection in memory.
- LINQ expressions return an enumeration, and by default the expression executes when you iterate through it using a foreach, but you can force it to iterate sooner using `.ToList()` or `.ToArray()`.



I hope this helps some people understand a little more about the difference between the levels of abstraction between a List and an IEnumerable. There is a lot more to learn from this topic, but this is a tricky starting point and one that took me a little while to get my head around. 





