# Pulse
A lightweight, ultra-modular C++ ECS framework for building flexible and efficient entity systems.

This is primarily as much an architectural challenge as it is a syntax challenge. This project served as a way for me to get a little more comfortable with ECS designs, C++ syntax, and generic templates and typing.

*That was back then; today, I'm more than comfortable with that kind of syntax, it's actually quite easy*
Small note, if you asked me to write this from memory without making any mistakes, I want to tell you right away that I would be incapable of it, the syntax is far too complex and I am only in my 2nd year of programming so I still have a long way to go before being able to do this professionally. However, I think I am clearly capable of creating minimal ecs systems that work very well. This is mainly an architectural experiment.

```cpp
// Copyright (c) October 2025 Félix-Olivier Dumas. All rights reserved.
// Licensed under the terms described in the LICENSE file

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

class Event /* BASE */ {
    /* Exemples de dérivés :
         - OnDamageEvent(...)
         - OnCollisionEvent(...)
    */

    /* event = collisionTrigger.generateEvent(entity) */
};

class Trigger /* BASE */ {
private:
        /* Exemples de dérivés :
             - collisionTrigger(...)
             - damageTrigger(...)
        */

public:

    /* event = collisionTrigger.generateEvent(entity) */

};

template<typename E = Event, typename S = System> class EventDispatcher {
private:
    std::map<E, std::vector<std::unique_ptr<S>>> eventListeners;

public:
    template <typename SS, typename E> void registerEvent(const E& event) {
        static_assert(std::is_base_of_v<S, SS>, "Must derive from System");
        static_assert(std::is_default_constructible_v<SS>, "Must be default constructible");

        auto& vec = eventListeners[e];
        for (auto& s : vec)
            if (typeid(*s) == typeid(SS))
                throw std::runtime_error("Component already exists!");
        vec.push_back(std::make_unique<SS>());
    }

    /* Dans le fond, c'est un registry ecs mais qui sert aussi de proxy entre
       les triggers, events et les systèmes qui appliquent le comportement. */

    /* Remove Event() */

    /* dispatcher.queue(event) */



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



    printEntity(registry, cat);

    if (registry.getComponent<Name>(dog)->name == registry.getComponent<Name>(cat)->name) {
        std::cout << "Identiques" << std::endl;
    } else std::cout << "Non-identiques" << std::endl;
   

    /*=======================================================*/
    
    /* Les composants fournissent les données (data-only) */
    /* Les systèmes définissent le comportement */

    /*
        class LookAtSystem : SystemBase() {
           override fun execute() { ... }
    */

    /* [DEPRECATED] Faire une liste internet d'entités (registered_entities? ou whatever)
       puis ensuite à chaque "update" ou appel, on incrit les entités qui présentent
       un type spécifique donné ou requis au système (<T?>) afin de "sort" en tant que
       tel les entités qui présentent les composantes requises. Ensuite, à chaque "update"
       on vérifie si les entités sont déja inscrites dans le registre interne (std::vector?) 
       puis s'il ne le sont pas, on les ajoute tout simplement. La logique ici serait de faire
       quelque chose qui évite d'une facon ou d'une autre la copie des éléments qui peut être 
       très couteuse en condition du nombre d'entités dans notre système. Enfin, il ne suffira
       que d'affecter les entités de notre collection à notre comportement voulu afin qu'il se
       "propage" au système en entier. À noter que ce système est global et ne présente pas encore
       de notion de "trigger" ou bien "d'évènement" alors les comportement seront appliqués globalement
       sans différenciation ou variation apparente. Écrit le lundi 20 octobre 2025 */

    /* 
        [Game Loop]
           |
           v
        [CollisionSystem] --checks--> Entities (player, monsters)
           |
           v
        Trigger détecté? Oui --> [PlayerHitMonsterEvent(player, monster)]
           |
           v
        [Event Dispatcher] ---> notifie [HealthSystem]
           |
           v
        [HealthSystem] --> applique damage sur player uniquement
    */

    /* 
        Rôle principal (Dispatcher)

        Prendre un event généré par un trigger.

        Identifier tous les systèmes abonnés à ce type d’event.

        Notifier chaque système en appelant sa méthode type onEvent(event).

        Optionnel : vider ou gérer une queue pour traiter tous les events accumulés dans la frame.
    */

    /*
        [EventDispatcher.dispatch()]
           └─ prend event depuis queue ou directement passé
           └─ regarde map<EventType, list<System*>>
           └─ pour chaque system dans la liste
                 └─ system.onEvent(event)
    */

    /*
        Trigger
        │
        ├─ condition(entity) : bool           # Vérifie si le trigger est activé pour une entité
        ├─ eventType : EventType             # Type d’event que ce trigger génère
        ├─ generateEvent(entity) : Event     # Crée l’événement associé à l’entité
        ├─ targetComponents : list<ComponentType>  # Optionnel : composants requis pour évaluer le trigger
    */

    /*
        [Trigger.condition(entity)] --true--> [Trigger.generateEvent(entity)]
                |
                v
        [EventDispatcher.queue(event)] --> [Systems abonnés traitent event]

    */

    /*
        System.update():
        for entity in managedEntities:
            if collisionTrigger.condition(entity):
                event = collisionTrigger.generateEvent(entity)
                dispatcher.queue(event)   # Trigger ne connaît pas les systèmes abonnés

    */

    /* Donc en conclusion, le plan est assez simple. Premièrement, il nous faudra créer les classes suivantes :
       System (déja prétent), Event, Trigger, EventDispatcher. Maintenant, nous avons un système qui dans notre
       architecture présente, doit avoir une fonction update() qui s'occupe de scanner les entités, les filtrer
       pour ensuite appliquer une condition (Trigger) qui si vrai, crée un évènement (Event) qui est ensuite envoyé
       au EventDispatcher, à noter que cet évènement contient les éléments du petit tableau ci-dessus. Ensuite,
       nous aurons aussi la fonction onEvent(event) qui, par l'intermédiaire du EventDispatcher, va recevoir des
       évnènements (Event envoyés par EventDispatcher) et appliquer le comportement voulu sur eux. Pour continuer, 
       nous avons ce fameux EventDispatcher qui, comme inscrit en haut, aura un dictionnaire (std::map) contenant
       un évènement (clé Event) associé à une liste de Systems auxquels sont abonnés à cet évent (Liste Systems, valeur)
       et représenté de tel -> std::vector<std::unique_ptr<Systems>> ou un équivalent. Lorsque ce EventDispatcher est notifié
       par un trigger, il vérifie premièrement si l'évènement est déja inscrit dans la liste, ni non, il l'ajoute, et puis 
       il parcours toute la liste de systems abonnés en appellant sys.onEvent(Event en question) afin d'appliquer le comportement
       voulu. De plus, il serait aussi possible d'implémenter un système de queue ou de buffer d'évènement afin de pouvoir
       traiter plusieurs évènements par frame sans surcharger le System ou bien le EventHandler, qui ne peut présentement
       que traiter un seul évent par appel. Écrit le lundi 20 Octobre 2025 par author.
       */

    /*=======================================================*/



    //newRegistry.addComponent<Component>(cat);
    //newRegistry.addComponent<Component>(cat);
    //newRegistry.addComponent<Component>(cat);

    //registry.addComponent<ComponentType pas instanciation>(cat); // possiblement faire ca, faut prendre le type de instancier dans la fonction addComponent();
    // Aussi, faire en sorte qu'on ne puisse qu'ajouter un seul composant de type par entité, par exemple uniquement un seul MockComponent...

    

    //auto comp = reg.getComponent<MockComponent>(cat); lui
}
```

## Output
```console
--------- Entity ---------
Name           : cat
Type           : class Entity
ID             : 0

---- Core Attributes ----
Name           : Garfield
Weight         : 10 Kg
Position       : (25, 75)
Velocity       : (5, 15)

------- Components -------
Count          : 4
Names          : Name, Weight, Velocity, Position
```
