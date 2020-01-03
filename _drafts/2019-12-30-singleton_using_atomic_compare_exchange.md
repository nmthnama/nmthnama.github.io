---
layout: post
title:  "Singleton class using atomic::compare_exchange_strong()"
date:   2019-12-30
desc: "weak reference in csharp"
keywords: "cpp,function pointer"
categories: [CSharp]
tags: [CSharp, weak reference]
icon: icon-html
---


```cpp
template <class Type>
class TDynamicSingleton
{
public:
	/**
	 * @brief Construct a new TDynamicSingleton object
	 * 
	 */
	TDynamicSingleton() {}
	virtual ~TDynamicSingleton() {}

public:
	/**
	 * @brief Get the Instance object
	 * 
	 * @return Type& 
	 */
	inline static Type& GetInstance()
	{
		// 멀티 스레드에 안전해야 한다.
		// 여기서 thread unsafe 하면 프로그래밍 할 때, 매우 골치 아파 진다.
		//static TSharedPtr<Type> m_instance;
		//static CAtomic m_staticAtomicCase;

		static std::shared_ptr<Type> m_instance;
		static std::atomic<int32_t> m_staticAtomicCase;

		// 캐싱
		if (m_instance != nullptr)
		{
			return *m_instance;
		}

		int comparand = 0;
		int newValue = 0;

		// staticAtomicCase 값에 따른 상황
		// 0 - 아직 생성 (시도) 조차 안됨
		// 1 - 어떤 스레드에서 생성 중
		// 2 - 제대로 생성됨
		//
		// 2 의 값이 된다면 해당 Type 은 안전하게 생성이 되어 막 가져와도 된다.
		// 그 이후의 thread safe 작업은 사용자의 몫
		do
		{
			comparand = 2;
			newValue = 2;
			if (m_staticAtomicCase.compare_exchange_strong(comparand, newValue) == true) // 제대로 생성 된 경우
			{
				// 객체가 이미 존재 한다 그러므로 그냥 break
				break;
			}

			comparand = 0;
			newValue = 1;
			if (m_staticAtomicCase.compare_exchange_strong(comparand, newValue) == true) // 생성 시도 조차 안된 경우
			{
				m_instance.reset(new Type());

				// 생성 됬다는걸 condition value 에 입력 후, break
				comparand = 1;
				newValue = 0;
				m_staticAtomicCase.compare_exchange_strong(comparand, newValue);
				break;
			}

			// 이곳에 오면 다른 스레드에서 생성중이란 뜻이다.
			while (true)
			{
				if (m_staticAtomicCase.load() == 2)
				{
					break;
				}
				std::this_thread::sleep_for(std::chrono::milliseconds(10));
			}
		} while (false);

		// 이곳에 나오면 위쪽에 instance 가 정상적으로 생성 됬다는걸 보장 해야 한다.
		if (m_instance == nullptr)
		{
			throw std::exception("Can not create singleton instance!");
		}

		return *m_instance;
	}
};
```

> atomic 
> - [http://egloos.zum.com/sweeper/v/3059861](http://egloos.zum.com/sweeper/v/3059861)

> atomic::compare_exchange_strong()
> - [http://www.cplusplus.com/reference/atomic/atomic/compare_exchange_strong/](http://www.cplusplus.com/reference/atomic/atomic/compare_exchange_strong/)

> static 지역 변수
> - [https://boycoding.tistory.com/169](https://boycoding.tistory.com/169)

> Template Dynamic Singleton
> - [https://vallista.tistory.com/entry/1-Singleton-Pattern-in-C](https://vallista.tistory.com/entry/1-Singleton-Pattern-in-C)

