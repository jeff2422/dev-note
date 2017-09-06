---
title: 观察者模式( Observer Pattern )
description: 观察者模式定义了对象之间的一对多以来，当一个对象改变状态时，它的所有依赖者都会收到通知并自动更新。
categories:
 - 设计模式
tags:
 - 设计模式
---

# 观察者模式

> 文章中的内容几乎全部来自于（《Head First Design Pattern》一书），稍做整理形成记录。
>
> 文章中涉及到的所有代码，均来自于[https://github.com/bethrobson/Head-First-Design-Patterns](https://github.com/bethrobson/Head-First-Design-Patterns)；强烈建议阅读此书（购买时，请选择正版）。

## 定义观察者

观察者模式定义了对象之间的一对多以来，当一个对象改变状态时，它的所有依赖者都会收到通知并自动更新。

## 类图

后续添加

## 气象站设计和实现

#### 主题接口

    public interface Subject {
    	public void registerObserver(Observer o);
    	public void removeObserver(Observer o);
    	public void notifyObservers();
    }
    
#### 在WeatherData中实现主题借口

    public class WeatherData implements Subject {
    	private ArrayList<Observer> observers;
    	private float temperature;
    	private float humidity;
    	private float pressure;
    	
    	public WeatherData() {
    		observers = new ArrayList<Observer>();
    	}
    	
    	public void registerObserver(Observer o) {
    		observers.add(o);
    	}
    	
    	public void removeObserver(Observer o) {
    		int i = observers.indexOf(o);
    		if (i >= 0) {
    			observers.remove(i);
    		}
    	}
    	
    	public void notifyObservers() {
    		for (Observer observer : observers) {
    			observer.update(temperature, humidity, pressure);
    		}
    	}
    	
    	public void measurementsChanged() {
    		notifyObservers();
    	}
    	
    	public void setMeasurements(float temperature, float humidity, float pressure) {
    		this.temperature = temperature;
    		this.humidity = humidity;
    		this.pressure = pressure;
    		measurementsChanged();
    	}
    	
    	public float getTemperature() {
    		return temperature;
    	}
    	
    	public float getHumidity() {
    		return humidity;
    	}
    	
    	public float getPressure() {
    		return pressure;
    	}
    
    }
    
#### 天气预报公告板借口

    public class CurrentConditionsDisplay implements Observer, DisplayElement {
    	private float temperature;
    	private float humidity;
    	private Subject weatherData;
    	
    	public CurrentConditionsDisplay(Subject weatherData) {
    		this.weatherData = weatherData;
    		weatherData.registerObserver(this);
    	}
    	
    	public void update(float temperature, float humidity, float pressure) {
    		this.temperature = temperature;
    		this.humidity = humidity;
    		display();
    	}
    	
    	public void display() {
    		System.out.println("Current conditions: " + temperature 
    			+ "F degrees and " + humidity + "% humidity");
    	}
    }
    
#### 启动气象站

    public class WeatherStation {
    
    	public static void main(String[] args) {
    		WeatherData weatherData = new WeatherData();
    	
    		CurrentConditionsDisplay currentDisplay = 
    			new CurrentConditionsDisplay(weatherData);
    		StatisticsDisplay statisticsDisplay = new StatisticsDisplay(weatherData);
    		ForecastDisplay forecastDisplay = new ForecastDisplay(weatherData);
    
    		weatherData.setMeasurements(80, 65, 30.4f);
    		weatherData.setMeasurements(82, 70, 29.2f);
    		weatherData.setMeasurements(78, 90, 29.2f);
    	}
    }
    
（未完待续……）
## Java内置的观察者模式

## 利用内置的支持重做气象站

## JDK中的其他应用
