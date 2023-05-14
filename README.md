# improved-octo-waffle
Aqui está um exemplo de código de qualidade para o desenvolvimento de sistemas de tempo real em C++
#include <iostream>
#include <chrono>
#include <thread>

// Define a estrutura do evento
struct Event {
    int id;
    std::chrono::steady_clock::time_point timestamp;
};

// Define a classe do agendador
class Scheduler {
public:
    // Adiciona um evento à lista de eventos
    void addEvent(Event e) {
        events.push_back(e);
    }

    // Executa a lista de eventos
    void run() {
        while (!events.empty()) {
            // Ordena a lista de eventos pelo tempo de timestamp
            std::sort(events.begin(), events.end(),
                [](const Event& a, const Event& b) { return a.timestamp < b.timestamp; });

            // Espera até o próximo evento acontecer
            auto wait_time = std::chrono::duration_cast<std::chrono::milliseconds>(events.front().timestamp - std::chrono::steady_clock::now());
            if (wait_time > std::chrono::milliseconds::zero()) {
                std::this_thread::sleep_for(wait_time);
            }

            // Executa o evento
            Event e = events.front();
            std::cout << "Executando evento " << e.id << " no tempo " << std::chrono::duration_cast<std::chrono::milliseconds>(e.timestamp.time_since_epoch()).count() << std::endl;
            events.erase(events.begin());
        }
    }

private:
    std::vector<Event> events;
};

int main() {
    // Cria um agendador e adiciona alguns eventos
    Scheduler scheduler;
    scheduler.addEvent({1, std::chrono::steady_clock::now() + std::chrono::milliseconds(100)});
    scheduler.addEvent({2, std::chrono::steady_clock::now() + std::chrono::milliseconds(50)});
    scheduler.addEvent({3, std::chrono::steady_clock::now() + std::chrono::milliseconds(150)});

    // Executa o agendador
    scheduler.run();

    return 0;
}
