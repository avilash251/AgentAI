package com.example.demo.websocket;

import com.example.demo.service.TokenAgentService;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;
import org.springframework.web.socket.*;
import org.springframework.web.socket.handler.TextWebSocketHandler;

import java.io.IOException;
import java.util.HashMap;
import java.util.Map;

@Component
public class TokenWebSocketHandler extends TextWebSocketHandler {

    @Autowired
    private TokenAgentService tokenAgentService;

    private final ObjectMapper objectMapper = new ObjectMapper();

    @Override
    public void handleTextMessage(WebSocketSession session, TextMessage message) throws IOException {
        Map<String, Object> request = objectMapper.readValue(message.getPayload(), Map.class);
        String type = (String) request.get("type");

        if ("getToken".equalsIgnoreCase(type)) {
            String token = tokenAgentService.fetchAccessToken();
            Map<String, Object> response = new HashMap<>();
            response.put("type", "token");
            response.put("token", token);
            session.sendMessage(new TextMessage(objectMapper.writeValueAsString(response)));
        } else {
            session.sendMessage(new TextMessage("{\"type\": \"error\", \"message\": \"Invalid command\"}"));
        }
    }
}
