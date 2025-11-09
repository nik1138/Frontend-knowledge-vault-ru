# Комплексные упражнения

## Упражнение 11: Создание мини-приложения
Создайте небольшое приложение, объединяющее все изученные концепции:

```javascript
// Задание: Создайте приложение для управления контактами
class ContactManager {
  constructor() {
    this.contacts = [];
    this.nextId = 1;
  }
  
  // Добавление контакта
  addContact(name, phone, email) {
    if (!name || !phone) {
      throw new Error("Имя и телефон обязательны");
    }
    
    const contact = {
      id: this.nextId++,
      name: name.trim(),
      phone: phone.trim(),
      email: email ? email.trim() : "",
      createdAt: new Date()
    };
    
    this.contacts.push(contact);
    return contact;
  }
  
  // Поиск контактов
  findContacts(searchTerm) {
    if (!searchTerm) {
      return [...this.contacts];
    }
    
    const term = searchTerm.toLowerCase();
    return this.contacts.filter(contact => 
      contact.name.toLowerCase().includes(term) ||
      contact.phone.includes(term) ||
      contact.email.toLowerCase().includes(term)
    );
  }
  
  // Обновление контакта
  updateContact(id, updates) {
    const contact = this.contacts.find(c => c.id === id);
    if (!contact) {
      throw new Error("Контакт не найден");
    }
    
    Object.assign(contact, updates);
    return contact;
  }
  
  // Удаление контакта
  removeContact(id) {
    const index = this.contacts.findIndex(c => c.id === id);
    if (index === -1) {
      throw new Error("Контакт не найден");
    }
    
    return this.contacts.splice(index, 1)[0];
  }
  
  // Получение всех контактов
  getAllContacts() {
    return [...this.contacts];
  }
  
  // Статистика
  getStats() {
    return {
      total: this.contacts.length,
      withEmail: this.contacts.filter(c => c.email).length,
      createdToday: this.contacts.filter(c => 
        new Date(c.createdAt).toDateString() === new Date().toDateString()
      ).length
    };
  }
}

// Использование
const contacts = new ContactManager();

try {
  contacts.addContact("Иван Иванов", "+7 (123) 456-78-90", "ivan@example.com");
  contacts.addContact("Мария Петрова", "+7 (123) 456-78-91");
  contacts.addContact("Алексей Сидоров", "+7 (123) 456-78-92", "alex@example.com");
  
  console.log("Все контакты:", contacts.getAllContacts());
  console.log("Поиск 'Иван':", contacts.findContacts("Иван"));
  console.log("Статистика:", contacts.getStats());
  
  // Обновление контакта
  const firstContact = contacts.getAllContacts()[0];
  contacts.updateContact(firstContact.id, { email: "ivan.ivanov@example.com" });
  
  console.log("Обновленный контакт:", contacts.findContacts("Иван")[0]);
} catch (error) {
  console.error("Ошибка:", error.message);
}