# Pulse
A lightweight, ultra-modular C++ ECS framework for building flexible and efficient entity systems.

```cpp
#include <iostream>
#include <unordered_map>
#include <string>
#include <sstream>
#include <map>
#include <type_traits>
#include <cassert>
#include <iomanip>

#define VAR_NAME(x) #x
#define CLASS_NAME(x) typeid(x).name()

template<typename T> struct ID {
    T value;
    explicit ID(T v = 0) : value(v) {}
};

template <typename T> void printValue(const T& t) {
    //std::cout << t << std::endl;
    std::cout << "     Address: " << static_cast<const void*>(&t) << std::endl;
}

using EntityID = uint32_t;

class Entity {
private:
    static EntityID nuid;
           EntityID uid;
public:
    Entity() : uid(nuid++) { }
    EntityID getId() const { return uid; }

    bool operator==(const Entity& other) const { return uid == other.uid; }
    bool operator!=(const Entity& other) const { return uid != other.uid; }
    bool operator<(const Entity& other)  const { return uid <  other.uid; }

}; EntityID Entity::nuid = 0;

class Component {
private:

public:
    virtual ~Component() = default;
    void doStuff() { std::cout << "Mock Component check\n"; }

};

class Velocity : public Component { public: float vx, vy = 0; };

class Position : public Component { public: float x, y = 0; };

class Weight : public Component { public: float weight = 0; };

class Name : public Component { public: std::string name = "default"; };

template<typename E = Entity, typename C = Component> class Registry {
private:
    std::map<E, std::vector<std::unique_ptr<C>>> registry;


public:
    Registry() {}

    template <typename CC, typename E> void addComponent(const E& e) {
        static_assert(std::is_base_of_v<C, CC>, "Must derive from Component");
        static_assert(std::is_default_constructible_v<CC>, "Must be default constructible");

        auto& vec = registry[e];
        for (auto& comp : registry[e])
            if (typeid(*comp) == typeid(CC))
                throw std::runtime_error("Component already exists!");
        vec.push_back(std::make_unique<CC>());
    }
    
    template <typename CC, typename E> CC* getComponent(const E& e) {
        for (auto& comp : registry[e])
            if (auto derived = dynamic_cast<CC*>(comp.get()))
                return derived;
        return nullptr;
    }

    template<typename CC = Component> size_t countComponents(const E& e) {
        size_t count = 0;
        for (auto& comp : registry[e])
            if (auto derived = dynamic_cast<CC*>(comp.get())) ++count;
        return count;
    }

    template <typename E> bool hasComponents(const E& e) const { return !registry[e].empty(); }

    template <typename CC = Component> std::vector<std::string> getComponentNames(const E& e) {
        std::vector<std::string> cnvec;
        for (auto& comp : registry[e])
            if (auto derived = dynamic_cast<CC*>(comp.get()))
                cnvec.push_back(typeid(*derived).name());
        for (auto& name : cnvec) {
            const std::string prefix = "class ";
            if (name.rfind(prefix, 0) == 0)
                name = name.substr(prefix.size());
        } return cnvec;
    }
};

class System {
private:


public:
    virtual void update(Registry<>& registry, float dt) = 0;

};

class ConsolePrintingSystem : public System {
private:


public:
    void update(Registry<>& registry, float dt) override {

    }
};

template <typename E> void printEntity(Registry<>& reg, const E& e) {
    std::cout << std::left;
    std::cout << "--------- Entity ---------\n";
    std::cout << std::setw(15) << "Name" << ": " << VAR_NAME(e) << "\n";
    std::cout << std::setw(15) << "Type" << ": " << CLASS_NAME(e) << "\n";
    std::cout << std::setw(15) << "ID" << ": " << e.getId() << "\n\n";

    std::cout << "---- Core Attributes ----\n";

    if (auto name = reg.getComponent<Name>(e))
        std::cout << std::setw(15) << "Name" << ": " << name->name << "\n";

    if (auto weight = reg.getComponent<Weight>(e))
        std::cout << std::setw(15) << "Weight" << ": " << weight->weight << " Kg\n";

    if (auto pos = reg.getComponent<Position>(e))
        std::cout << std::setw(15) << "Position" << ": (" << pos->x << ", " << pos->y << ")\n";

    if (auto vel = reg.getComponent<Velocity>(e))
        std::cout << std::setw(15) << "Velocity" << ": (" << vel->vx << ", " << vel->vy << ")\n";

    auto componentNames = reg.getComponentNames(e);
    std::cout << "\n------- Components -------\n";
    std::cout << std::setw(15) << "Count" << ": " << componentNames.size() << "\n";

    std::cout << std::setw(15) << "Names" << ": ";
    for (size_t i = 0; i < componentNames.size(); ++i) {
        if (i > 0) std::cout << ", ";
        std::cout << componentNames[i];
    }
    std::cout << "\n\n\n\n";
}

int main() {
    auto registry = Registry<Entity, Component>();

    auto cat     = Entity();
    auto dog     = Entity();
    auto hamster = Entity();

    registry.addComponent<Name>(cat);
    registry.addComponent<Weight>(cat);
    registry.addComponent<Velocity>(cat);
    registry.addComponent<Position>(cat);

    registry.addComponent<Name>(dog);
    registry.addComponent<Weight>(dog);

    registry.addComponent<Name>(hamster);

    auto name     = registry.getComponent  <Name>  (cat);
    auto weight   = registry.getComponent <Weight> (cat);
    auto velocity = registry.getComponent<Velocity>(cat);
    auto position = registry.getComponent<Position>(cat);

    auto componentCount = registry.countComponents(cat);
        
    name->name = "Garfield";
    weight->weight = 10.0f;
    velocity->vx = 5.0f; velocity->vy = 15.0f;
    position->x = 25; position->y = 75;



    printEntity(registry, hamster);

    if (registry.getComponent<Name>(dog)->name == registry.getComponent<Name>(cat)->name) {
        std::cout << "Identiques" << std::endl;
    } else std::cout << "Non-identiques" << std::endl;
   


    


    //newRegistry.addComponent<Component>(cat);
    //newRegistry.addComponent<Component>(cat);
    //newRegistry.addComponent<Component>(cat);

    //registry.addComponent<ComponentType pas instanciation>(cat); // possiblement faire ca, faut prendre le type de instancier dans la fonction addComponent();
    // Aussi, faire en sorte qu'on ne puisse qu'ajouter un seul composant de type par entité, par exemple uniquement un seul MockComponent...

    

    //auto comp = reg.getComponent<MockComponent>(cat); lui
}
```
