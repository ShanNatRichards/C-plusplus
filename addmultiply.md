# SinglyLinkedList Calculator

## Create an algorithm that reads floating point numbers from a txt file and then adds/multiplies these floats.
## The Use of the C++ Math Library Is Not Allowed
### Each number must be built using a linked list. For e.g. the number 489.59 would be represented by list of linked nodes like this: 4-> 8 -> 9 ->. ->5 -> 9

### Big O Analysis
For additions, the algortihm runs in O(N). For multiplications, algorithm runs in O(N^2).



### Namespaces Used

```C++

#include <iostream>
#include <fstream>
#include <cstdlib>
using namespace std;

```
###  Structs 

```C++

struct qnode 
{ 
  // Node used to create a queue to represent a number. So the number 357 in the queue looks like  (3<- 5<- 7)
  int data;  // node's member used to store a single integer character
  qnode * next; //pointer to the next node in the queue.
};

struct snode 
{  
  // Node used to create a stack to represent a number. So the number 357 in the stack looks like  (7<- 5<- 3)
  int data; // node's member used to store a single integer character
  snode * prev; //pointer to the next node in the stack.
};


struct dnode
{ 
  // Holds a reference for either a queue or stack of numbers, and defines whether the number is floating point and position of decimal point.
  bool isFloat; //Is the number a floating point? 
  int decimalpos; //The position of decimal point. 
  snode * head; // Pointer to the head of the stack
  qnode * head_q; //Pointer to the head the queue.
};
```

### Main Function

```C++
int main (int argc, char * argv[]) 
{

  if (fileExists(argv[2]) && fileExists(argv[3]))
  {
     char * file1 = argv[2];
     char * file2 = argv[3];
     dnode *const l1 = getNums(file1);
     dnode *const l2 = getNums(file2);
     char * command = argv[1];
     dnode * res = NULL;

     if (strcmp(command, "add") == 0)
     {
        res = add (l1, l2);
        printList(res);
    
    } else if (strcmp(command, "multiply") == 0)
    {
       res = multiply (l1, l2);
       printList(res);
    } else 
    {
       cout << "Command is not right - type 'add' or 'multiply'." <<endl;
    }
  }
return 0;
}

```

## Functions for Addition:

### Add Function
   ###Adds 2 dnode structs passed in as arguments.

```C++
//Facade for the addition functionality, returns datalist with the result of the addition.

dnode * add(dnode * a, dnode * b ){
  if (a == NULL || b == NULL)
  return NULL;

  qnode * result = addstarter(a, b);
  int num = reverseAndCount(result);
  int decimal_places = (a ->decimalpos > b->decimalpos)? a -> decimalpos : b -> decimalpos;

  dnode * c = new dnode;
  c -> isFloat = (decimal_places > 0);

  if (c -> isFloat)
  c -> decimalpos = num - decimal_places;
  else
  c -> decimalpos = 0;

  c ->head_q = result;
  return c;
}

```

### Addition SubFunctions:

```C++

qnode * addstarter(dnode * a, dnode * b) {
  //Manages the adding of numbers sitting behind the decimal point.
  qnode * sum = new qnode;
  sum -> next = NULL;
  sum -> data = 0;

  qnode * const summation_head = sum;
  snode * sa = a -> head;
  snode * sb = b -> head;


  if (a ->isFloat || b->isFloat) {
    int decimal_place_for_a = a -> decimalpos;
    int decimal_place_for_b = b -> decimalpos;

    if (decimal_place_for_a > decimal_place_for_b)
        copyToSum(sa, sum, (decimal_place_for_a - decimal_place_for_b));
    else if (decimal_place_for_b > decimal_place_for_a)
        copyToSum(sb, sum, (decimal_place_for_b - decimal_place_for_a));
  }

  addhandler(sa,sb,sum, 0);
  return summation_head;

}

```

```C++
// Recursive function is responsible for adding the numbers with matching place positions in the dnode
// For example, if the numbers we want to add are 3876 and 123, the function adds 876 to 123.

void addhandler(snode * a, snode * b, qnode * &sum, int carry_val) 
{

  if ( a == NULL && b == NULL && carry_val == 0)
    return;

  if (sum == NULL){
    sum = new qnode;
    sum -> next = NULL;
  }

  if (a == NULL) {
    handleLeftOvers(b, sum, carry_val);
    return;
  }

  if (b== NULL) {
    handleLeftOvers (a, sum, carry_val);
    return;
  }

  int sum_value = a -> data + b ->data + carry_val;
  sum -> data = sum_value % 10;
  carry_val = sum_value / 10;

  addhandler(a->prev, b -> prev, sum->next, carry_val);
}

```

```C++

// Recurive function responsible for placing the left-over values to the front of the list for the sum.
// Let's say we're adding 12325 and 68, then this function inserts 123 after addhandler function adds '25' and '68'.

void handleLeftOvers(snode * &t , qnode * &sum, int carry_val) 
{

  if (t == NULL && carry_val == 0)
    return;

  if (sum == NULL) {
    sum = new qnode;
    sum -> next = NULL;
  }

  if (t == NULL && carry_val > 0) {
    sum -> data = carry_val;
    return;
  }

  int sum_value = t->data + carry_val;
  sum -> data = sum_value % 10;
  carry_val = sum_value / 10;

  handleLeftOvers(t -> prev, sum -> next, carry_val);

}

```

```C++
//Let's say we're adding 2.8976 and 2.1, then this function is responsible
//for inserting '976' (the unmatched place positions) into the end of the list for the sum. 

void copyToSum(snode*&t, qnode *&sum, int repeat) 
{

  if (repeat == 0) return;

  sum -> data = t -> data;

  sum -> next = new qnode;
  sum = sum -> next;
  sum -> next = NULL;
  t = t-> prev;

  copyToSum(t, sum, --repeat);
}

```



## Functions for Multiplication:

### Function: Multiply
    ### returns a dnode strcut with the multiplication result
    
``` C+++
dnode * multiply(dnode * a, dnode * b) 
{
   qnode * head = NULL;
   dnode * d_top = NULL;
   int num = 0;
   head = multiplystarter(a, b);

   if (head) {
     num = reverseAndCount(head);
     d_top = new dnode;
     d_top -> decimalpos = num - (a -> decimalpos + b -> decimalpos);
     d_top -> isFloat = (d_top -> decimalpos > 0);
     d_top -> head_q = head;
  }

  return d_top;
}

```
### Multiply Sub Functions:

```C++

qnode * multiplystarter (dnode * da, dnode * db) 
{
  //This function uses a while loop to make sure that we handle the '0' displacement needed for each iteration of the multiplication.
  if (da == NULL || db == NULL)
    return NULL;

  qnode * const product_head = new qnode;
  product_head-> data = 0;
  product_head-> next = NULL;
  snode *snode_b = db -> head;
  snode *snode_a = da -> head;
  int level = 0; //variable keeps track of of the displacement needed for multiplication

  while (snode_b) {
    qnode * product = product_head;
    int multiplier = snode_b -> data;

    for (int i = 0; i < level; i++) { //find the right spot to start adding
      product = product -> next;
    }
    
    multiplyhelper(multiplier, snode_a, product, 0);
    snode_b = snode_b -> prev;
    level++;
  }

  return product_head;
}

```

```C++
void multiplyhelper(const int b, snode * a, qnode * &c, int carry_val) 
{
  //Recursive function that manages addition in the multiplication.   
  if (a == NULL && carry_val == 0)
  return;

  if (c == NULL) {
    c = new qnode;
    c -> data = 0;
    c ->next = NULL;
  }

  if (a == NULL) {
    c -> data = carry_val;
    return;
  }
  else {
    int multiply_math = b * a -> data + carry_val + c -> data;
    c -> data = multiply_math % 10;
    carry_val = multiply_math / 10;
    multiplyhelper(b, a->prev, c->next, carry_val);
  }
}

```

```C++
int reverseAndCount(qnode * &a) 
{
   //When printing out multiplication result, reverse the list and return the count of the number of integer characters present.
   qnode * headnew = NULL;
   qnode * temp = NULL;
   int numcount = 0;

   while (a) {
     temp = a;
     a = a -> next;
     temp -> next  = headnew;
     headnew = temp;
     numcount++;
   }

   a = headnew;
   return numcount;
}

```

## General Functions:

### Function: Check if file exists

```C++
// Function to check if the file we're trying to read from exists.

bool fileExists(char * name) 
{
  ifstream file (name);

  if(!file.is_open()) {
    cout << "Oops. That file called " << name <<  " doesn't seem to exist. Try again."<<endl;
    return false;
  }
  file.close();
  return true;

}
```

### Function: Read from the file and return a dnode struct

```C++

dnode * getNums (char * name) 
{

  ifstream file(name);
  char tempdata = '0';
  int dec_counter = -1;
  snode * sn = NULL;
  dnode * top = NULL;

  while (file) {
    file >>tempdata;

    if (file.eof())
      break;

    if (tempdata == '.') {
      dec_counter++;
      continue;
    }

    if (dec_counter > -1)
      dec_counter++;

    snode * temp = new snode;
    temp -> data  = tempdata - '0';
    temp -> prev = sn;
    sn = temp;
  }

  if (sn) {
    top = new dnode;

    if (dec_counter > -1) {
    top -> isFloat = true;
    top -> decimalpos = dec_counter;
  }else {
    top -> isFloat = false;
    top -> decimalpos = 0;
  }
    top -> head = sn;
  }

  return top;
}
```

### Functions: Return the position of the decimal point
   
```C++
//Return the position of the decimal point if number is a float.

int getDecimalPos(dnode * head) {
  return head -> decimalpos;
}

```
### Functions:  Return if the number is a floating point

```C++
//Return true or false if data list contains a floating point number.

bool read(dnode * head) { 
  return head -> isFloat;
}
```

### Function : Print Linkedlist

```C++

void printList(dnode * res) 
{

  qnode * result = res -> head_q;
  int dec_counter = 0;
  int decimal_pos = getDecimalPos(res);

   while(result) {
     cout <<result -> data << endl;
     dec_counter++;
     if (dec_counter == decimal_pos)
        cout << '.' << endl;

    result = result -> next;
  }
}
```


