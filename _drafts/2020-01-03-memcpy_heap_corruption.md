---
layout: post
title:  "Usage of function pointer int cpp"
date:   2020-01-03
desc: "function pointer in cpp"
keywords: "cpp,memcpy,heap corruption"
categories: [CPP]
tags: [cpp, function pointer]
icon: icon-html
---

- src 가 할당된 크기보다 큰 memory 를 dest 에 복사하는 경우 heap corruption 발생
- `pData` 는 src 의 포인터인데, src 는 메모리 할당을 패킷 받은 크기인 <mark>30</mark> 만큼만 되어 있던 상태<br>
- memcpy_s 호출 시, `복사할 길이`에 해당하는 `sizeof(A_Packet)` 은 <mark>30,000</mark><br>
- dest 인 `temp` 로 할당되지 않은 메모리 `29,970(=30,000-30)`가 memcpy 되면서 heap corruption 발생<br>
``` cpp
A_Packet* A_Packet::Deserialize(void* pData) {
    A_Packet* packet = reinterpret_cast<A_Packet*>(pData);
    char temp[sizeof(A_Packet)];
    memcpy_s(temp, sizeof(A_Packet), pData, (sizeof(A_Packet) + 
                0 ));
    unsigned short index = HEADER_SIZE;
    GetT(temp, packet->m_Va, index);
    GetT(temp, packet->m_Vb, index);
    GetT(temp, packet->m_Vc, index);
    GetT(temp, packet->m_Vd, index);
    GetT(temp, packet->m_Ve, index);
    GetT(temp, packet->m_Vf, index);
    return packet;
}
```
- 너무 당연히 문제가 되는 코드인데, 내가 만든 데이터버퍼(패킷 받은 만큼만 메모리 할당)를 개발자가 만든 deserialize 에 사용하는 상황 때문에 문제를 확인하기 더 어려웠음<br>
- 어차피 GetT() 함수에서 사용하는건 쓰레기 값이 아니므로 문제가 없을 것이라 착각함 <br>