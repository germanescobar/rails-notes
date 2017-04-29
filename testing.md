## Configurar Selenium con Chrome

```
self.use_transactional_tests = false

Capybara.register_driver :selenium do |app|
  Capybara::Selenium::Driver.new(app, browser: :chrome)
end
```
