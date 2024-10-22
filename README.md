from FastAPI import FastAPI, HTTPException
from pydantic import BaseModel
from typing import Dict

app = FastAPI()

# Модели данных
class Service(BaseModel):
    name: str
    description: str
    price: float
    duration: int
    image_url: str = None

class Specialist(BaseModel):
    name: str
    avatar_url: str = None

class Order(BaseModel):
    client_name: str
    service_id: int
    specialist_id: int
    time: str

# Данные
services: Dict[int, Service] = {}
specialists: Dict[int, Specialist] = {}
orders: Dict[int, Order] = {}

service_id_counter = 1
specialist_id_counter = 1
order_id_counter = 1

# Эндпоинты CRUD для услуг
@app.post("/services/")
def create_service(service: Service):
    global service_id_counter
    services[service_id_counter] = service
    service_id_counter += 1
    return {"id": service_id_counter - 1, "service": service}

@app.get("/services/{service_id}")
def read_service(service_id: int):
    if service_id in services:
        return services[service_id]
    raise HTTPException(status_code=404, detail="Service not found")

@app.put("/services/{service_id}")
def update_service(service_id: int, service: Service):
    if service_id in services:
        services[service_id] = service
        return service
    raise HTTPException(status_code=404, detail="Service not found")

@app.delete("/services/{service_id}")
def delete_service(service_id: int):
    if service_id in services:
        del services[service_id]
        return {"detail": "Service deleted"}
    raise HTTPException(status_code=404, detail="Service not found")

# Эндпоинты CRUD для специалистов
@app.post("/specialists/")
def create_specialist(specialist: Specialist):
    global specialist_id_counter
    specialists[specialist_id_counter] = specialist
    specialist_id_counter += 1
    return {"id": specialist_id_counter - 1, "specialist": specialist}

@app.get("/specialists/{specialist_id}")
def read_specialist(specialist_id: int):
    if specialist_id in specialists:
        return specialists[specialist_id]
    raise HTTPException(status_code=404, detail="Specialist not found")

@app.delete("/specialists/{specialist_id}")
def delete_specialist(specialist_id: int):
    if specialist_id in specialists:
        del specialists[specialist_id]
        return {"detail": "Specialist deleted"}
    raise HTTPException(status_code=404, detail="Specialist not found")

# Эндпоинты CRUD для заказов
@app.post("/orders/")
def create_order(order: Order):
    global order_id_counter
    if not specialist_is_available(order.specialist_id, order.time):
        raise HTTPException(status_code=403, detail="Specialist is not available at this time")
    orders[order_id_counter] = order
    specialists[order.specialist_id].appointments.append(order.time)
    order_id_counter += 1
    return {"id": order_id_counter - 1, "order": order}

@app.get("/orders/{order_id}")
def read_order(order_id: int):
    if order_id in orders:
        return orders[order_id]
    raise HTTPException(status_code=404, detail="Order not found")

@app.delete("/orders/{order_id}")
def delete_order(order_id: int):
    if order_id in orders:
        specialist_id = orders[order_id].specialist_id
        time = orders[order_id].time
        specialists[specialist_id].appointments.remove(time)
        del orders[order_id]
        return {"detail": "Order deleted"}
    raise HTTPException(status_code=404, detail="Order not found")

# Проверка доступности специалиста
def specialist_is_available(specialist_id: int, time: str):
    if specialist_id in specialists:
        return time not in specialists[specialist_id].appointments
    return False

# Запуск сервера
if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
