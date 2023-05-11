---
title:  "[JPA]save & saveAll"
excerpt: "Difference of save and saveAll in JPA"

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2023-05-11
---


## save & saveAll

---


### save

```java
@Override
public Nft create(NftDto nftDto) {
    Nft nft = nftDto.toEntity();
    repository.save(nft);
    return nft;
}
```

- 단일 entity의 저장에 사용
- entityManager.persist(entity) or repository.save(entity)
- entity가 이미 영속성 컨텍스트에 존재(1차 캐싱)하면 update
- entity가 영속성 컨텍스트에 없으면 새로운 entity를 영속성 컨텍스트에 추가하고 DB에 저장

<br />

---
### saveAll

```java
@Override
public List<Nft> createAll(List<NftDto> nftDto) {
    List<Nft> nfts = new ArrayList<>();
    for(int i =0; i < nftDto.size(); i++){
        nfts.add(nftDto.get(i).toEntity());
    }
    repository.saveAll(nfts);
    return nfts;
}
```

- 한 번에 여러 entity의 저장에 사용
- entityManager.merge(entity) or repository.saveAll(entities)
- 일부의 entity가 이미 영속성 컨텍스트에 존재(1차 캐싱)하면 update
- 일부 entity가 영속성 컨텍스트에 없으면 새로운 entity를 영속성 컨텍스트에 추가하고 DB에 저장

<br />

---
### Processing speed & Difference

<br />

> 여러 개의 entity를 save로 처리했을 때와 saveAll로 처리했을 때

```java
//save
@Override
public List<Nft> createAll(List<NftDto> nftDto) {
    for(int i =0; i < nftDto.size(); i++){
        repository.saveAll(nftDto.get(i).toEntity());
    }
    return nfts;
}

//saveAll
@Override
public List<Nft> createAll(List<NftDto> nftDto) {
    List<Nft> nfts = new ArrayList<>();
    for(int i =0; i < nftDto.size(); i++){
        nfts.add(nftDto.get(i).toEntity());
    }
    repository.saveAll(nfts);
    return nfts;
}
```

> saveAll이 save보다 성능이나 여러 면에서 유용한 이유

- saveAll은 단일 transaction을 사용하여 여러 개의 entity를 한 번에 저장(네트워크 오버헤드 감소)
- saveAll은 여러 개의 entity를 처리할 때 최적화된 쿼리를 생성(처리 속도 상승)
- saveAll은 여러 개의 entity를 한 번에 저장하여 영속성 컨텍스트의 상태 변화를 최소화함(dirty checking 최적화)

<br />

---


