

# Factory method pattern
two major varieties of implementation
1. subclasses to define an implementation,because there's no reasonable default.
2. parameterized factory methods.lets the factory method create multiple kinds
of products.The factory takes a parameter that identifies the kind of object to create.
3. c++ template


```
//the second case
class Creator{
    public:
    virtual Product* Create(ProductId);
    };
Product* Creator::Create(ProductId){
    if(id==MINE)return new MyProduct;
    if(id==YOURS)return new YourProduct;
    //repeat for remaining products...
    return 0;
}

Product* MyCreator::Create(ProductId id){
    if(id==YOURS)return new MyProduct;
    return Creator::Create(id);
}

```

# Prototype
just call clone constructor in Clone()
```
class Door:public MapSite{
    public:
    Door{};
    // copy constructor
    Door(const Door&);
    virtual void Initialize(Room*,Room*);
    virtual Door* Clone() const;
}

void Door::Clone()const{
    return new Door(*this);
}
```

# singleon


# adaptor(give a completely new interface)
two approaches

1. multiple inheritance to adapt interface
2. composition to combine classes with different interfaces
In this approach , the adapter maintains a pointer to adapee
# bridge

# composite
leaf and composite
# decorator(don't change its interface)
take stream for example.
Stream: MemeoryStream,FileStream,StreamDecorator
StreamDecorator:ASCII7Stream,CompressingStream
```
class StreamDecorator:public Stream{
    public:
    StreamDecorator(Stream* stream){
        stream_=stream;
        }
    private:
    Stream* stream_;// track of stream
    };
```
# facade
expose subsystem
# flyweight
intrinct(shared)
extrinct(not shared)


# chain of resposibility
just like linked list
```
class Base{
    public:
    void Add(Base* n){
        if(next){
            next->Add(n);
            }else{
                next = n;
            }

        }
    virtual void Handle(int i){
        next->Handle(i);
        }
    };
// subclass use forwarding mode
```

# command
sperate receiver and command
```
class Command{
    public:
    Command(Receiver* receiver,Action method){
        }
        void Execute(){
            (receiver->*method)();
        }
};
```

# interpreter
Context is for recoding the value of variable
Expression is for executing calculation

# iterator
set friend class Iterator iter of class arggregation;

# mediator
interconnect with mediator rather than objects

# observer
subject has a list of all observers, observers will update if something happen

# state
machine change the behavior with the variance of state in it
machine has  a state point to record the current state

# visitor
double dispatch

