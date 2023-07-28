package com.mtb.iap.common.controller;


import com.mtb.iap.common.model.AccountsFilter;
import com.mtb.iap.common.model.User;
import lombok.RequiredArgsConstructor;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.reactive.function.client.WebClient;
import reactor.core.publisher.Mono;


@RestController
@RequiredArgsConstructor
@RequestMapping("/v3")
public class AccountsController {

  private final WebClient.Builder webClientBuilder;

  @Value("${cbs.accounts.rest.uri}")
  private String accountsUri;


  @GetMapping("/accounts")
  public Mono<ResponseEntity<String>> getAccountSummary(
          @RequestHeader("MTBToken") String jwtToken,
          @RequestParam("filter") AccountsFilter accountFilter) {

    System.out.println("hit");

    return webClientBuilder.build()
            .get()
            .uri(accountsUri)
            .header("MTBToken", jwtToken)
            .retrieve()
            .bodyToMono(String.class)
            .map(data -> ResponseEntity.ok().body(data));
  }

}
